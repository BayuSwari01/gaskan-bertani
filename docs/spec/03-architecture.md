# §3 Architecture

## §3.1 Layout

```
index.html  gallery.html
src/main.ts                 boot: WebGL2 check -> app/game -> (debug builds) debug API
src/env.d.ts                declare const __GB_DEBUG__: boolean; declare const __GB_VERSION__: string
src/core/                   PURE game logic: no three, no DOM, no Date.now, no Math.random, no timers
  env.d.ts                  declare function structuredClone<T>(v: T): T
  types.ts result.ts clock.ts constants.ts clone.ts prng.ts time.ts
  data/catalogTypes.ts data/catalog.ts data/validateCatalog.ts
  levels.ts water.ts grid.ts state.ts invariants.ts
  rules/{goods,plots,trees,animals,supplies,well,mill,market,placement,levelUp,aid}.ts
  commands.ts selectors.ts
  save/{schema,migrations,serialize}.ts   save/__fixtures__/v0-basic.json
  testing/builders.ts       test-only helpers
src/app/                    options, systemClock, persistence, store, uiState, interaction,
                            settings, tabGuard, game (composition root)
src/render/                 renderer, scene, camera, coords, picking, lighting, ground, scenery,
                            sceneSync, view, iconTexture
  input/{gestures,pointerInput}.ts
  layers/{bubbles,blobShadows,ghost,highlight,grid}.ts
  anim/{tweens,effects,wander,scheduler}.ts
  models/{kit,palette,materials,library,plot,overlays,decorations}.ts
  models/crops/{tomat,padi,cabai,jagung,singkong,bawangMerah}.ts
  models/trees/{treeKit,jeruk,mangga,rambutan,kelapa,durian}.ts
  models/animals/{ayam,kambing,sapi}.ts
  models/buildings/{rumah,gudang,penggilingan,sumur}.ts
src/icons/{atlas,draw}.ts   Canvas2D icons shared by render and ui
src/audio/sfx.ts            WebAudio-synthesized sound effects
src/ui/                     dom, strings.id, format, ports, viewModels, hud, toolbar, modeChip, toast,
                            floatingText, tooltip, panel, placementBar, hints, overlay, styles.css
  panels/{seedPicker,objectInfo,shop,storage,mill,well,settings}.ts
  modals/{confirm,levelUp,withered}.ts
src/debug/{debugApi,types,fixtures}.ts   loaded only when __GB_DEBUG__
src/gallery/main.ts
e2e/support/{test,gb,snap}.ts  e2e/gl-profiles.json  e2e/sNN-*.spec.ts  e2e/sNN-*.mobile.spec.ts
scripts/{run,doctor,verify-bundle,verify-hygiene}.mjs
docs/README.md              index and reading order of the documentation
docs/spec/NN-*.md           this specification, one file per section (§1–§12)
docs/game-design.md  docs/visual-guide.md  docs/glossary.md  docs/research-farmville2.md
docs/decisions.md           append-only decision log
```

## §3.2 Layers

| From | May import |
|---|---|
| core | core only |
| icons | core (types only) |
| audio | nothing in `src` |
| render | core, icons |
| ui | core, icons |
| app | core, render, ui, icons, audio |
| debug, gallery | everything |

`main.ts` imports `debug` only through `if (__GB_DEBUG__) await import('./debug/debugApi')`.

## §3.3 Enforcement

- **TypeScript configs.**
  - `tsconfig.base.json`: `target`/`lib` ES2023, `module` ESNext, `moduleResolution` bundler, `strict`,
    `noUncheckedIndexedAccess`, `noImplicitOverride`, `noFallthroughCasesInSwitch`,
    `verbatimModuleSyntax`, `erasableSyntaxOnly`, `isolatedModules`, `resolveJsonModule`,
    `skipLibCheck`, `noEmit`, `types: []`.
  - `tsconfig.json` extends it with lib `[ES2023, DOM, DOM.Iterable]` and types
    `["vite/client", "node"]`; it includes `src`, `e2e` and the `*.config.ts` files.
  - `tsconfig.core.json` extends it with lib `[ES2023]` and types `[]`; it includes `src/core` and
    excludes `**/*.test.ts` and `src/core/testing`. Any DOM use in core is then a compile error.
  - TypeScript 6 defaults: `types` is `[]`, so list `vite/client` and `node` explicitly; `strict` is on;
    `baseUrl` and `moduleResolution: node` are deprecated.
- **ESLint** (`eslint.config.js`, flat config with `defineConfig` from `eslint/config`):
  - `ignores`: `node_modules`, `dist`, `dist-e2e`, `.tmp`, `.npm-cache`, `test-artifacts`,
    `coverage`, `.vitest`, `.agent-logs`, `prompts`.
  - `js.configs.recommended` and `tseslint.configs.recommendedTypeChecked` (`projectService: true`);
    `tseslint.configs.disableTypeChecked` for `**/*.{js,mjs}`.
  - Rules: `consistent-type-imports`, `no-floating-promises`, `no-eval`, `no-implied-eval`,
    `no-new-func`, `no-restricted-properties` for `Math.random`.
  - `no-restricted-syntax` selectors: `MemberExpression[property.name=/^(innerHTML|outerHTML)$/]`,
    `CallExpression[callee.property.name='insertAdjacentHTML']`,
    `CallExpression[callee.property.name='setAttribute'][arguments.0.value='style']`.
  - `linterOptions.reportUnusedDisableDirectives: 'error'`.
  - One `no-restricted-imports` block per row of §3.2.
  - The core block bans the globals `window document localStorage sessionStorage navigator
    performance requestAnimationFrame setTimeout setInterval fetch`, and `Date.now`, `Math.random`
    and `new Date()`.
  - ESLint rule options do not merge across config objects, so the core block must repeat the
    security selectors.

## §3.4 Data flow

```
pointer/touch -> render/input/gestures: tap | pan | pinch | wheel | paint
  pan/pinch/wheel -> camera rig -> requestRender
  tap(point)   -> app/interaction: picker.pick(point) -> mode handler / selectors.primaryAction
  paint(point) -> app/interaction: the stroke's action on each newly entered target (§8.3)
     -> store.dispatch(cmd) -> core.applyCommand(state, cmd, now, catalog): Result<{state, events}>
        ok:  store.state = next; persistence.schedule(); notify {kind: 'dispatch', events}
        err: notify {kind: 'rejected', error, detail}
subscribers: render/sceneSync.sync(state, now) -> meshes, overlays, bubbles, shadows -> requestRender
             ui: hud, floating text, toasts, level-up modal, open panel refresh; app: sound effects
1 Hz app tick (not in e2e): store.tick() -> rebase check -> notify {kind: 'tick'} -> resync visuals
```

## §3.5 Core types (authoritative names; `src/core/types.ts`)

```ts
export type ObjectId = string; // matches OBJECT_ID_RE = /^o[1-9]\d{0,8}$/
export type CropId = 'tomat' | 'padi' | 'cabai' | 'jagung' | 'singkong' | 'bawang_merah';
export type TreeId = 'jeruk' | 'mangga' | 'rambutan' | 'kelapa' | 'durian';
export type AnimalId = 'ayam' | 'kambing' | 'sapi';
export type BuildingId = 'rumah' | 'gudang' | 'penggilingan' | 'sumur';
export type GoodId =
  | 'tomat' | 'gabah' | 'cabai' | 'jagung' | 'singkong' | 'bawang_merah'
  | 'jeruk' | 'mangga' | 'rambutan' | 'kelapa' | 'durian'
  | 'telur' | 'susu_kambing' | 'susu_sapi';

export interface CropState { defId: CropId; plantedAt: number; wateredAt: number | null }
export interface PlotObject {
  id: ObjectId; kind: 'plot'; x: number; y: number; crop: CropState | null; fertilized: boolean;
}
export interface TreeObject {
  id: ObjectId; kind: 'tree'; defId: TreeId; x: number; y: number;
  plantedAt: number; harvests: number; wateredAt: number | null; fertilized: boolean;
}
export interface AnimalObject {
  id: ObjectId; kind: 'animal'; defId: AnimalId; x: number; y: number; boughtAt: number;
  adult: boolean; bottlesTaken: number; bottledAt: number | null; fedAt: number | null;
}
export interface BuildingObject {
  id: ObjectId; kind: 'building'; defId: BuildingId; x: number; y: number;
  collectedAt: number | null; // sumur only; always null for the other buildings
}
export type FarmObject = PlotObject | TreeObject | AnimalObject | BuildingObject;

export interface GameState {
  version: 1;
  createdAt: number; lastSeenAt: number; nextId: number;
  rng: number;                                   // uint32 state of the core PRNG (pupuk rolls)
  coins: number; xp: number;                     // integers; the level is derived from xp
  water: { amount: number; updatedAt: number };  // §4.6
  feed: number; fertilizer: number; bottles: number; // pakan, pupuk, Botol Susu
  goods: Partial<Record<GoodId, number>>;        // Gudang inventory; no zero entries
  objects: Record<ObjectId, FarmObject>;         // key === value.id
  aidClaimedAt: number | null;
}

export type Command =
  | { type: 'plow'; x: number; y: number }
  | { type: 'plant'; plotId: ObjectId; cropId: CropId }
  | { type: 'water'; targetId: ObjectId }          // plot or tree
  | { type: 'harvest'; targetId: ObjectId }        // plot or tree
  | { type: 'clearPlot'; plotId: ObjectId }        // withered crop only, free
  | { type: 'revive'; plotId: ObjectId }           // withered crop only, costs koin
  | { type: 'fertilize'; targetId: ObjectId }      // plot or tree
  | { type: 'buyTree'; treeId: TreeId; x: number; y: number }
  | { type: 'buyAnimal'; animalId: AnimalId; x: number; y: number }
  | { type: 'buyBottles'; qty: number }
  | { type: 'feed'; targetId: ObjectId }           // baby: 1 Botol Susu; adult: pakan
  | { type: 'collectWell'; targetId: ObjectId }
  | { type: 'sell'; goodId: GoodId; qty: number }
  | { type: 'mill'; goodId: GoodId; qty: number }
  | { type: 'move'; objectId: ObjectId; x: number; y: number }
  | { type: 'remove'; objectId: ObjectId }
  | { type: 'claimAid' };

export type ErrorCode =
  | 'NOT_FOUND' | 'WRONG_TARGET' | 'OUT_OF_BOUNDS' | 'OCCUPIED' | 'LIMIT_REACHED'
  | 'NOT_ENOUGH_COINS' | 'NOT_ENOUGH_WATER' | 'NOT_ENOUGH_FEED' | 'NOT_ENOUGH_BOTTLES'
  | 'NOT_ENOUGH_FERTILIZER' | 'NOT_ENOUGH_GOODS' | 'FEED_FULL' | 'LEVEL_LOCKED'
  | 'PLOT_EMPTY' | 'PLOT_NOT_EMPTY' | 'ALREADY_WATERED' | 'ALREADY_FERTILIZED' | 'NOT_READY'
  | 'WITHERED' | 'NOT_WITHERED' | 'NOT_HUNGRY' | 'NOT_MILLABLE' | 'INVALID_QTY'
  | 'NOT_REMOVABLE' | 'AID_UNAVAILABLE' | 'UNSUPPORTED';

export type ErrorDetail = Readonly<Record<string, string | number>>;
export type Result<T, E = ErrorCode> =
  | { ok: true; value: T }
  | { ok: false; error: E; detail?: ErrorDetail };

export type GameEvent =
  | { type: 'coins' | 'xp' | 'water' | 'feed' | 'fertilizer' | 'bottles'; delta: number; objectId?: ObjectId }
  | { type: 'goods'; goodId: GoodId; delta: number; objectId?: ObjectId }
  | { type: 'levelUp'; level: number; rewardCoins: number; rewardWater: number;
      unlocked: ReadonlyArray<{ kind: 'crop' | 'tree' | 'animal'; id: string }> }
  | { type: 'levelBonus'; ripened: number; revived: number }
  | { type: 'objectAdded' | 'objectRemoved'; objectId: ObjectId }
  | { type: 'objectChanged'; objectId: ObjectId; reason: 'planted' | 'watered' | 'harvested'
      | 'cleared' | 'revived' | 'fertilized' | 'fed' | 'grewUp' | 'collected' | 'moved' | 'ripened' };

export type ObjectStatus =
  | 'empty' | 'needsWater' | 'growing' | 'ready' | 'withered' | 'hungry' | 'cooldown' | 'idle';
export type VisualState =
  | { kind: 'plot'; fertilized: boolean; crop: null | { id: CropId; stage: 0 | 1 | 2 | 3 | 4 | 5 } }
      // stage 0 seeded (dry soil), 1-3 growing, 4 ripe, 5 withered
  | { kind: 'tree'; id: TreeId; stage: 0 | 1 | 2 | 3; fruit: 'none' | 'unripe' | 'ripe'; fertilized: boolean }
  | { kind: 'animal'; id: AnimalId; adult: boolean }
  | { kind: 'building'; id: BuildingId; full?: boolean }; // full: sumur only
export type Bubble =
  | null | { kind: 'water' } | { kind: 'ready'; goodId: GoodId } | { kind: 'withered' }
  | { kind: 'bottle' } | { kind: 'feed' } | { kind: 'well' };
export type PanelId = 'seed' | 'objectInfo' | 'shop' | 'storage' | 'mill' | 'well' | 'settings';
export type PrimaryAction =
  | { kind: 'plant' | 'water' | 'harvest' | 'feed' | 'collectWell' | 'withered' | 'info' }
  | { kind: 'openPanel'; panel: PanelId };
export interface ObjectView {
  id: ObjectId; status: ObjectStatus; visual: VisualState; bubble: Bubble;
  remainingMs: number | null; progress: number | null;
}
export interface ScreenPoint { x: number; y: number; visible: boolean; occluded: boolean } // CSS px
export type DispatchResult =
  | { ok: true; events: GameEvent[] } | { ok: false; error: ErrorCode; detail?: ErrorDetail };
export type StoreChange =
  | { kind: 'dispatch'; cmd: Command; events: GameEvent[] }
  | { kind: 'rejected'; cmd: Command; error: ErrorCode; detail?: ErrorDetail }
  | { kind: 'tick' }
  | { kind: 'replace'; reason: 'load' | 'reset' | 'fastForward' | 'rebase' };
```

**`applyCommand(state, cmd, now, catalog)`** does, in order:

1. `deepClone` the state into a draft.
2. `rebaseIfClockWentBack(draft, now)` (§3.7).
3. Run the handler, which validates, mutates the draft and pushes events. A rejected command returns
   its error and the original state is untouched.
4. `applyLevelUps` (§4.2): rewards for every crossed level, then the level-up bonus once.
5. `draft.lastSeenAt = max(draft.lastSeenAt, now)`.
6. `checkInvariants(draft)`; a violation throws (programmer error).
7. Return `{ ok: true, value: { state: draft, events } }`.

Rules never throw for player mistakes; they return error codes.

## §3.6 Store, clock and UI ports

- `Clock { now(): number }` returns epoch ms and may jump backwards. `ManualClock` (core) provides
  `now/advance/set` with explicit fields (`erasableSyntaxOnly` forbids parameter properties).
  `SystemClock` lives in `app/` and uses `Date.now`.
- `Store` (`app/store.ts`): `getState()`; `now()` (effective time: rebased, monotonic);
  `dispatch(cmd): DispatchResult`; `subscribe(fn: (c: StoreChange) => void): () => void`; `tick()`
  (updates `lastSeenAt`, emits `tick`); `replaceState(s, reason)`; `flushSave()`. Autosave is
  debounced to 400 ms after each successful dispatch.
- UI ports (`ui/ports.ts`), because ui may not import app:
  ```ts
  type PlaceItem = { kind: 'tree'; id: TreeId } | { kind: 'animal'; id: AnimalId };
  type Mode = 'normal' | 'seed' | 'plow' | 'fertilize' | 'move' | 'sell' | 'place';
  interface UiState {
    mode: Mode; seedCropId?: CropId; placeItem?: PlaceItem; movingId?: ObjectId;
    ghost?: { x: number; y: number; valid: boolean; reason?: ErrorCode };
    painting: boolean; openPanel: PanelId | null; panelArgs?: { plotId?: ObjectId; objectId?: ObjectId };
    selectedId: ObjectId | null;
  }
  interface UiDeps {
    store: StoreLike; catalog: Catalog;
    projector: { projectObject(id: ObjectId): ScreenPoint | null; projectTile(x: number, y: number): ScreenPoint };
    modes: { enterSeed(cropId: CropId): void; enterTool(tool: 'plow' | 'fertilize' | 'move' | 'sell'): void;
             enterPlace(item: PlaceItem): void; enterMove(id: ObjectId): void; cancel(): void };
    uiState: { get(): UiState; subscribe(fn: (s: UiState) => void): () => void;
               openPanel(id: PanelId, args?: UiState['panelArgs']): void; closePanel(): void };
  }
  ```
  `app/uiState.ts` and `app/interaction.ts` implement them.

## §3.7 Time model

- Core functions receive `now` (epoch ms) as a parameter. Absolute times are fields named `*At`
  (plus `water.updatedAt`); durations end in `Ms`.
- State stores facts (timestamps of actions). Every status, stage, bubble and timer is derived from
  `(state, now)` by selectors. Nothing that time alone would change is stored, so there is no per-tick
  simulation and offline progress (including withering) is automatic.
- Clock rollback: if `now < state.lastSeenAt`, `rebaseIfClockWentBack` shifts every timestamp by
  `now − lastSeenAt`. This keeps every remaining duration: nothing freezes and nothing is gained.
- Debug fast-forward: `fastForward(ms)` is `shiftTimestamps(state, −ms)` and survives reloads.
- Each producer holds at most one pending output (a ripe crop, a ready tree, a full well), so large
  forward jumps are harmless.

## §3.8 Worked examples (informative)

These traces show how the layers cooperate. They add no new rules; the normative parts are §3.4–§3.7,
§4 and §8.

**A. Clicking a ripe tomato (normal mode).** Plot `o7` holds a Tomat whose `wateredAt` is 70 s ago
(`growMs` is 60 s, so it is ripe).

1. `pointerInput` feeds `pointerdown` to `gestures`. The recognizer calls `canPaintAt(point)`;
   `app/interaction` picks `o7`, sees through `canHarvestPlot` that it is ripe, and answers `true`
   (a harvest stroke could start here).
2. The pointer goes up within the slop and 600 ms, so `gestures` emits `tap` instead of a stroke.
3. `interaction.onTap(point)`: `picker.pick(point)` returns `{ objectId: 'o7' }`;
   `selectors.primaryAction(state, o7, now, catalog)` returns `{ kind: 'harvest' }`; the interaction
   calls `store.dispatch({ type: 'harvest', targetId: 'o7' })`.
4. `store.dispatch` reads `now` from its clock and calls `applyCommand`. `rules/plots.harvestPlot`
   checks the phase (ripe), adds 1 `tomat` through `rules/goods.addGoods` (2 if the plot was
   fertilized), adds `xpHarvest`, clears `crop` and `fertilized`, and pushes the events
   `goods +1 (o7)`, `xp +1 (o7)`, `objectChanged harvested (o7)`. `applyLevelUps` runs (no level
   crossed here), `lastSeenAt` is updated and `checkInvariants` passes.
5. The store replaces its state, schedules the save (400 ms) and notifies
   `{ kind: 'dispatch', cmd, events }`.
6. Subscribers react: `sceneSync.sync` sees that the model key of `o7` changed from `plot:tomat:4` to
   `plot:empty` and swaps the geometry; the `ready` bubble disappears. The HUD re-renders koin and XP.
   `floatingText` shows "+1 Tomat" and "+1 XP" at `projector.projectObject('o7')`. From Sprint 14 the
   mesh squashes, a Tomat icon flies to the Gudang button and `sfx` plays `harvest`.
7. 400 ms later `persistence` writes the envelope to `gaskan-bertani:save`.

**B. A rejected command.** The player clicks a seeded plot while the HUD shows 0 air.
`primaryAction` returns `{ kind: 'water' }`; `applyCommand` returns
`{ ok: false, error: 'NOT_ENOUGH_WATER', detail: { need: 1, have: 0 } }` and the state object is
unchanged. The store notifies `{ kind: 'rejected', … }`, `toast` shows the §8.7 text "Air tidak cukup.
Tunggu air terisi atau ambil dari Sumur." and nothing else happens (no save, no render change).

**C. A drag-to-act stroke in seed mode.** The mode is `seed` with `seedCropId = 'padi'`. The pointer
goes down on an empty plot, so `canPaintAt` answers `true`; after moving beyond the slop `gestures`
emits `paintStart` and then `paint` for every move. For each point, `interaction` resolves the plot
under the pointer, skips plots already visited in this stroke or not empty, and dispatches
`plant(padi)` once per plot. When a dispatch fails with `NOT_ENOUGH_COINS`, the stroke ends, one toast
is shown and seed mode exits. The camera does not move during the stroke.

**D. Coming back after a night away.** The game was closed at 22:00 with Padi watered at 21:00
(`growMs` 4 j, `witherMs` 8 j). It is opened at 08:00. Nothing was simulated while closed:
`persistence` loads the save, `store` uses the current time, and the selectors derive
`e = 11 j ≥ growMs` and `e < growMs + witherMs` (12 j), so the plot is ripe with "layu dalam 1j 00m".
Had the player come back at 09:00 or later, the same derivation would give withered.
