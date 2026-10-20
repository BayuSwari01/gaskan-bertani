# TODO — Gaskan Bertani

Gaskan Bertani is a single-player browser game in Three.js low-poly 3D. It clones the core loop of
FarmVille 2: crops on plots, fruit trees and animals. All in-game text is Bahasa Indonesia; code,
comments, tests and docs are English.

How to use this file:

- `## Specification` (§1–§12) is the reference for every agent. It contains no tasks; never add
  checkboxes to it. Sprint tasks refer to it as §N.
- The sprints follow it. The coder works on exactly one `## Sprint N` per loop and checks off its
  tasks. The reviewer adds `### Fix` items under a sprint when something must be fixed.
- Balance numbers live only in `src/core/data/catalog.ts`, which mirrors §5. Tests read numbers from
  the catalog and never hard-code them.
- When the specification is silent or ambiguous, choose the simplest option that is consistent with
  it and append a dated one-line note to `docs/decisions.md`.

## Specification

### §1 Product scope

- **Game.** "Gaskan Bertani", single-player and offline in the browser. It clones the FarmVille 2
  core loop and nothing else:
  - Crops on plots: plow a plot (free) → plant a seed (koin) → water it (1 air) → it grows → ripe →
    harvest (goods + XP). A ripe crop that is left too long withers.
  - Fruit trees: buy a sapling → water it (N air) → it grows → fruit ready → harvest → water again.
  - Animals: buy a baby → give it Botol Susu (milk bottles) until it is adult → an adult gets hungry →
    feed it pakan → its product pops out immediately (+XP, chance of pupuk) → cooldown → hungry again.
- **Content** (ids in parentheses):
  - Crops (6): Tomat (`tomat`), Padi (`padi`, its good is Gabah `gabah`), Cabai (`cabai`),
    Jagung (`jagung`), Singkong (`singkong`), Bawang Merah (`bawang_merah`).
  - Trees (5): Jeruk (`jeruk`), Mangga (`mangga`), Rambutan (`rambutan`), Kelapa (`kelapa`),
    Durian (`durian`).
  - Animals (3): Ayam (`ayam`, baby "Anak Ayam", product Telur `telur`), Kambing (`kambing`, baby
    "Cempe", product Susu Kambing `susu_kambing`), Sapi (`sapi`, baby "Pedet", product Susu Sapi
    `susu_sapi`).
- **Supporting systems** (FarmVille 2 mechanics, rules in §4): koin (coins, earned only by selling
  goods); XP and levels 1–20 with unlocks and a level-up bonus; air (water) that regenerates plus a
  Sumur (well); pakan (feed) made from goods in the Penggilingan Pakan (feed mill); pupuk
  (fertilizer); Botol Susu bought with koin; the Gudang (barn: inventory and selling); drag-to-act over
  many plots; withering; "Bantuan Desa" (aid that prevents a soft lock).
- **Platform.** Desktop and mobile browsers at or above Vite 8's baseline (Chrome/Edge 111,
  Firefox 114, Safari 16.4). WebGL2 is required.
- **Look.** Low-poly 3D with flat shading and a bright cartoon palette, under a fixed isometric
  orthographic camera that never rotates. Every model is built in code from primitives; there are no
  model, texture, audio or font files.
- **Out of scope:** farm expansion, crafting/kitchen, mastery, groves, prized animals,
  neighbors/social, quests, premium currency, object rotation, store-to-inventory, backend, network,
  accounts, analytics, service worker, music.

### §2 Environment, tooling and commands

#### §2.1 Owner prerequisites (one-time, done by the human; agents cannot do these)

- Linux x86-64 or arm64: Debian 12/13 or Ubuntu 22.04/24.04/26.04. No GPU and no display needed.
- Node `^22.13.0 || ^24.0.0 || >=26.0.0` (24 LTS recommended) with npm 10 or newer, and git.
- Chromium OS libraries, once, as root: `sudo npx -y playwright@1.64.1 install-deps chromium`.
- Outbound internet access for npm packages and the Chromium download.
- To view the game from another machine, use an SSH tunnel, for example
  `ssh -L 5317:127.0.0.1:5317 <server>`, then open http://127.0.0.1:5317. Servers never bind to
  `0.0.0.0`.

#### §2.2 Sandbox rules (agents may write only inside the project folder)

- Run npm only from the project root. `.npmrc` sets `cache=.npm-cache`, which npm resolves relative
  to the current directory.
- Every tool runs through `node scripts/run.mjs <bin> [...args]`; the npm scripts already do this.
  The script:
  - sets `TMPDIR`, `TMP` and `TEMP` to `<root>/.tmp`, `HOME` to `<root>/.tmp/home`, and
    `XDG_CACHE_HOME` and `XDG_CONFIG_HOME` under that home, creating the folders;
  - sets `PLAYWRIGHT_BROWSERS_PATH=0`, so browsers live in `node_modules/playwright-core/.local-browsers`;
  - spawns `node_modules/.bin/<bin>` (or `process.execPath` when `<bin>` is `node`) with inherited
    stdio and exits with the child's exit code.
- Never run `npx playwright …` directly, never `npm i -g`, never `sudo`. Never write to `~` or `/tmp`.
- `npm ci` wipes `node_modules`, which removes the browsers; `pree2e` reinstalls them and does
  nothing when they are present.
- If Chromium fails with "Host system is missing dependencies", or cannot start inside the sandbox,
  write `BLOCKED.md` with the exact error and the §2.1 command.

#### §2.3 Config files

- `.npmrc`, exactly:
  ```
  cache=.npm-cache
  update-notifier=false
  fund=false
  audit=false
  save-exact=true
  ```
- `.gitignore`: keep every existing line (the my-ai-agent template ignores `run.sh`, `config.sh`,
  `prompts/`, `.agent-logs/` and others) and make sure these lines exist: `node_modules/`,
  `.npm-cache/`, `.tmp/`, `dist/`, `dist-e2e/`, `test-artifacts/`, `coverage/`, `.vitest/`, `*.log`.
- `.prettierignore`: the generated folders above plus `**/*.md`, `package-lock.json`,
  `.agent-logs/` and `prompts/`. Markdown is never formatted, so edits to `todo.md` and the reports
  cannot break `format:check`.
- `.prettierrc.json`: `{"singleQuote": true, "printWidth": 100, "trailingComma": "all"}`.
- `.editorconfig`: UTF-8, LF, 2-space indent, final newline.

#### §2.4 Dependencies (exact versions, nothing else)

- Runtime: `three@0.186.1`.
- Dev: `@eslint/js@10.0.1`, `@playwright/test@1.64.1`, `@types/node@22.20.5`,
  `@types/three@0.186.0`, `@vitest/coverage-v8@5.0.5`, `eslint@10.13.0`, `prettier@3.9.10`,
  `typescript@6.0.3`, `typescript-eslint@8.72.0`, `vite@8.3.4`, `vitest@5.0.5`.
- `package.json` has `"private": true`, `"type": "module"` and
  `"engines": {"node": "^22.13.0 || ^24.0.0 || >=26.0.0"}`.
- If an exact version cannot be installed, use the newest patch of the same minor version and log it
  in `docs/decisions.md`. Never use TypeScript 7.x: typescript-eslint 8.72 supports only TypeScript
  below 6.1.

#### §2.5 npm scripts

| Script | Command |
|---|---|
| dev | `node scripts/run.mjs vite` (127.0.0.1:5317; for humans) |
| build | `node scripts/run.mjs vite build` → `dist/` (no debug API, no gallery, CSP meta added) |
| build:e2e | `node scripts/run.mjs vite build --mode e2e` → `dist-e2e/` (debug API, gallery, CSP) |
| typecheck | `tsc -p tsconfig.json && tsc -p tsconfig.core.json` |
| lint | `eslint . --max-warnings=0` |
| format / format:check | `prettier --write .` / `prettier --check .` |
| test / test:coverage | `node scripts/run.mjs vitest run` / `node scripts/run.mjs vitest run --coverage` |
| e2e:install / pree2e | `node scripts/run.mjs playwright install chromium` / `npm run e2e:install` |
| e2e | `npm run build:e2e && node scripts/run.mjs playwright test` (one file: `npm run e2e -- e2e/s06-camera.spec.ts`) |
| verify | `node scripts/verify-bundle.mjs && node scripts/verify-hygiene.mjs` |
| check | typecheck, lint, format:check, test, build, verify (fast; no browser) |
| check:all | `npm run check && npm run e2e` |
| doctor | `node scripts/run.mjs node scripts/doctor.mjs` |

Ports: 5317 (dev) and 4317 (preview), both with `strictPort`. If a port is busy, run
`pkill -f "vite preview"` (and `pkill -f chrome-headless-shell` if needed). Always stop any server you
start; `npm run e2e` manages its own server.

#### §2.6 Vite config (`vite.config.ts`)

- `mode` is `production` or `e2e`; `debug = mode !== 'production'`.
- `define: { __GB_DEBUG__: JSON.stringify(debug), __GB_VERSION__: JSON.stringify(<package.json version>) }`.
- `server` and `preview` use `host: '127.0.0.1'`, ports 5317 and 4317, `strictPort: true`.
- `build.outDir` is `dist-e2e` in e2e mode, otherwise `dist`. `build.sourcemap: debug`.
- `build.rolldownOptions.input`: `index.html`, plus `gallery.html` only when `debug`. Build the paths
  with `fileURLToPath(new URL('./x.html', import.meta.url))`.
- A build-only plugin (`apply: 'build'`) adds this CSP meta tag through `transformIndexHtml`
  (`injectTo: 'head-prepend'`): `default-src 'self'; script-src 'self'; style-src 'self'; img-src
  'self' data: blob:; connect-src 'self'; object-src 'none'; base-uri 'none'; form-action 'none'`.

### §3 Architecture

#### §3.1 Layout

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
docs/spec.md (pointer to this file)  docs/decisions.md (append-only decision log)
```

#### §3.2 Layers

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

#### §3.3 Enforcement

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

#### §3.4 Data flow

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

#### §3.5 Core types (authoritative names; `src/core/types.ts`)

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

#### §3.6 Store, clock and UI ports

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

#### §3.7 Time model

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

### §4 Game rules

#### §4.1 Grid and objects

- The farm is 24×24 tiles. Grid `(x, y)` has its origin at the back corner; tile `(x, y)` covers world
  `[x, x+1] × [y, y+1]` on the XZ plane (grid y maps to world z).
- Footprints: plot 1×1, tree 2×2, animal per definition (ayam 1×1, kambing 2×2, sapi 2×2), building
  per definition. An object's `(x, y)` is the minimum corner of its footprint.
- Objects must be in bounds and must not overlap. At most `MAX_OBJECTS = 300` objects.
- Buildings: exactly one of each, created by `newGame`. They can be moved but never removed.

#### §4.2 XP, levels, limits and the level-up bonus

- `level` = the highest L with `levels[L-1].xp ≤ xp`, capped at 20. XP keeps accumulating after 20.
- XP sources (everything else gives 0 XP): watering a plot `xpWaterPlot`; watering a tree the tree's
  `xpWater`; fertilizing `xpFertilize`; harvesting a crop or tree its `xpHarvest`; giving a baby a
  bottle `xpBottle`; feeding an adult its `xpFeed`. XP is clamped at `XP_MAX`.
- Level gates: planting a crop and buying a tree or animal need `level ≥ unlockLevel`
  (`LEVEL_LOCKED`, `detail.level`).
- Plot limit: `plotLimit(level) = min(plotBase + plotPerLevel × (level − 1), plotMax)`; plowing fails
  with `LIMIT_REACHED` (`detail.kind = 'plots'`, `detail.limit`) when the player already owns that many.
- Level-up: after every successful command, for each level crossed in order, add that level's
  `rewardCoins` and `rewardWater` (water may exceed `waterRegenCap`, clamped at `waterHardCap`) and
  emit `levelUp` with the items whose `unlockLevel` equals that level.
- Level-up bonus (FarmVille 2), applied once when at least one level was crossed:
  - every plot whose crop has been watered and is growing or withered gets `wateredAt = now − growMs`
    (ripe now with a full wither window); count `ripened` (was growing) and `revived` (was withered);
  - every tree that is watered and growing gets `wateredAt = now − cycleMs` (fruit ready now);
  - emit `levelBonus { ripened, revived }` plus `objectChanged` with reason `ripened` or `revived` for
    each affected object. Seeded (unwatered) crops, animals and the well are not affected.

#### §4.3 Crops and plots

- Phase of a plot, derived from `(plot, now)`; with `e = now − wateredAt`, `g = growMs`,
  `w = witherMs`:
  - empty: `crop === null`;
  - seeded (stage 0, needs water): `crop.wateredAt === null`;
  - growing: `e < g`; stage `1 + floor(3e / g)` (1–3); `remainingMs = g − e`; `progress = e / g`;
  - ripe (stage 4): `g ≤ e < g + w`; `remainingMs = g + w − e` (time until it withers);
  - withered (stage 5): `e ≥ g + w`.
- `plow`: free, 0 XP; creates `{ crop: null, fertilized: false }`.
- `plant`: costs `seedCost`; sets `crop = { defId, plantedAt: now, wateredAt: null }`; 0 XP.
- `water` on a plot: costs 1 air; sets `wateredAt = now`; `+xpWaterPlot`.
- `harvest` on a plot: ripe only; adds `yield` (+1 when the plot is fertilized) of the crop's product
  to the goods; `+xpHarvest`; then `crop = null`, `fertilized = false`.
- `clearPlot`: withered only; `crop = null`, `fertilized = false`; free, 0 XP.
- `revive`: withered only; costs `reviveMultiplier × seedCost`; sets `wateredAt = now − growMs`
  (ripe now with a full wither window); `fertilized` is kept; 0 XP.
- `fertilize` on a plot: allowed when the plot is empty, seeded, growing or ripe (not withered) and
  not yet fertilized; costs 1 pupuk; `+xpFertilize`; the flag waits for the next harvest.

#### §4.4 Trees

- A tree is watered once per cycle: needs water (`wateredAt === null`) → `water` costs
  `waterPerCycle` air at once (`+xpWater`) → growing for `cycleMs` → ready → `harvest` adds
  `fruitYield` (+1 when fertilized) of its fruit (`+xpHarvest`), `harvests += 1`, `wateredAt = null`,
  `fertilized = false` → needs water again. Trees never wither.
- Visual, with `p = (now − wateredAt) / cycleMs` while growing:
  - first cycle (`harvests === 0`): needs water → stage 0 (sapling); growing `p < 1/3` → stage 0,
    `p < 2/3` → stage 1, otherwise stage 2; ready → stage 3 with fruit `ripe`;
  - later cycles: always stage 3; fruit `none` while needing water or growing with `p < 0.5`,
    `unripe` while growing with `p ≥ 0.5`, `ripe` when ready.
- `fertilize` on a tree: allowed when not yet fertilized (any phase); costs 1 pupuk; `+xpFertilize`.
- `buyTree` creates `{ plantedAt: now, harvests: 0, wateredAt: null, fertilized: false }`.

#### §4.5 Animals

- Baby (`adult === false`): hungry when `bottledAt === null` or `now ≥ bottledAt + bottleIntervalMs`,
  otherwise cooldown. `feed` on a hungry baby costs 1 Botol Susu, `bottlesTaken += 1`,
  `bottledAt = now`, `+xpBottle`. When `bottlesTaken === bottlesToAdult` the animal becomes adult
  (`adult = true`, `fedAt = null`, so it is hungry immediately) and an `objectChanged` event with
  reason `grewUp` is emitted.
- Adult: hungry when `fedAt === null` or `now ≥ fedAt + cooldownMs`, otherwise cooldown. `feed` on a
  hungry adult costs `feedPerMeal` pakan, sets `fedAt = now`, adds `yield` of its product to the goods
  immediately and gives `+xpFeed`. Then it rolls pupuk: `mulberry32Step(state.rng)` gives a value in
  `[0, 1)` and the new rng state; if the value is below `fertilizerChance` and
  `fertilizer < fertilizerCap`, `fertilizer += 1` (event `fertilizer`).
- `buyAnimal` creates a baby `{ boughtAt: now, adult: false, bottlesTaken: 0, bottledAt: null,
  fedAt: null }`; at most `maxOwned` animals of each kind (`LIMIT_REACHED`, `detail.kind = 'animals'`).
- Bubbles: hungry baby → `bottle`; hungry adult → `feed`; otherwise none.

#### §4.6 Water (air)

- With `cap = waterRegenCap`: if `amount ≥ cap`, then `k = 0` and `current = amount`; otherwise
  `k = max(0, floor((now − updatedAt) / waterRegenMs))` and `current = min(cap, amount + k)`.
- `nextAt = current ≥ cap ? null : updatedAt + (k + 1) × waterRegenMs`.
- Spending `n` needs `current ≥ n` (else `NOT_ENOUGH_WATER`, `detail.need`, `detail.have`). Then
  `amount = current − n` and `updatedAt = current ≥ cap ? now : updatedAt + k × waterRegenMs` (partial
  progress is kept).
- Adding `n` (well, level-up): first settle as above (`amount = current`, `updatedAt` as for a spend of
  0), then `amount = min(waterHardCap, amount + n)`.
- The HUD shows `current/cap` (for example "12/30", or "45/30" above the cap) and a "+1 mm:ss"
  countdown while below the cap.

#### §4.7 Sumur (well)

- Full when `collectedAt === null` or `now ≥ collectedAt + wellRefillMs`; otherwise refilling.
- `collectWell` on the sumur: full only (`NOT_READY`, `detail.ms`); adds `wellWater` air (§4.6 add);
  `collectedAt = now`; 0 XP. Bubble `well` when full.

#### §4.8 Goods, selling, milling, bottles and aid

- Goods have no storage limit; each count is an integer between 1 and `GOOD_MAX` (absent when 0).
  An addition beyond `GOOD_MAX` fails with `LIMIT_REACHED` (`detail.kind = 'goods'`).
- `sell`: `qty` is an integer ≥ 1 (`INVALID_QTY`); removes the goods and adds `price × qty` koin
  (beyond `COINS_MAX`: `LIMIT_REACHED`, `detail.kind = 'coins'`); 0 XP.
- `mill`: only goods with `millFeed` (`NOT_MILLABLE`); adds `qty × millFeed` pakan, all or nothing
  against `feedCap` (`FEED_FULL` with `detail.max = floor((feedCap − feed) / millFeed)`).
- `buyBottles`: `qty` is an integer from 1 to 99; costs `qty × bottlePrice`; `bottles + qty` must not
  exceed `bottleCap` (`LIMIT_REACHED`, `detail.kind = 'bottles'`).
- Aid ("Bantuan Desa", the soft-lock guard) is available when all of these hold: koin below the
  cheapest seed cost among unlocked crops; no goods; no plot has a crop that is not withered; no tree
  is watered (growing or ready); `aidClaimedAt` is null or at least `aidCooldownMs` ago. `claimAid`
  adds `aidCoins` and sets `aidClaimedAt = now`; otherwise `AID_UNAVAILABLE`.

#### §4.9 Moving and removing

- `move` works for every kind, buildings included. The target footprint must be in bounds and free,
  ignoring the object itself. The object keeps its state.
- `remove` deletes the object. Buildings fail with `NOT_REMOVABLE`. Trees and animals refund
  `floor(cost × removeRefundPercent / 100)` koin; plots refund nothing and lose their crop.

#### §4.10 Command summary

Every command fails with `NOT_FOUND` for an unknown id and `WRONG_TARGET` for the wrong kind of
object; an unknown command type gives `UNSUPPORTED`. Level-ups are applied after every successful
command (§4.2).

| Command | Preconditions (error) | Effects | XP |
|---|---|---|---|
| plow | in bounds (OUT_OF_BOUNDS); tile free (OCCUPIED); plot limit and object limit (LIMIT_REACHED) | new empty plot | 0 |
| plant | crop is null (PLOT_NOT_EMPTY); unlocked (LEVEL_LOCKED); koin (NOT_ENOUGH_COINS) | −seedCost; crop planted, unwatered | 0 |
| water (plot) | has a crop (PLOT_EMPTY); not withered (WITHERED); unwatered (ALREADY_WATERED); 1 air (NOT_ENOUGH_WATER) | −1 air; wateredAt = now | xpWaterPlot |
| water (tree) | unwatered (ALREADY_WATERED); waterPerCycle air (NOT_ENOUGH_WATER) | −waterPerCycle air; wateredAt = now | xpWater |
| harvest (plot) | has a crop (PLOT_EMPTY); not withered (WITHERED); ripe (NOT_READY, detail.ms) | +yield goods (+1 if fertilized); crop = null; fertilized = false | xpHarvest |
| harvest (tree) | ready (NOT_READY, detail.ms) | +fruitYield goods (+1 if fertilized); harvests + 1; wateredAt = null; fertilized = false | xpHarvest |
| clearPlot | has a crop (PLOT_EMPTY); withered (NOT_WITHERED) | crop = null; fertilized = false | 0 |
| revive | has a crop (PLOT_EMPTY); withered (NOT_WITHERED); koin (NOT_ENOUGH_COINS) | −reviveMultiplier × seedCost; wateredAt = now − growMs | 0 |
| fertilize | plot not withered (WITHERED); not fertilized (ALREADY_FERTILIZED); 1 pupuk (NOT_ENOUGH_FERTILIZER) | −1 pupuk; fertilized = true | xpFertilize |
| buyTree | unlocked; koin; in bounds; free; object limit | −cost; new tree | 0 |
| buyAnimal | unlocked; koin; maxOwned and object limit (LIMIT_REACHED); in bounds; free | −cost; new baby | 0 |
| buyBottles | qty 1–99 integer (INVALID_QTY); koin; bottleCap (LIMIT_REACHED) | −qty × bottlePrice; +qty bottles | 0 |
| feed (baby) | hungry (NOT_HUNGRY, detail.ms); 1 bottle (NOT_ENOUGH_BOTTLES) | −1 bottle; bottlesTaken + 1; grows up at bottlesToAdult | xpBottle |
| feed (adult) | hungry (NOT_HUNGRY, detail.ms); pakan (NOT_ENOUGH_FEED, detail.need) | −feedPerMeal; fedAt = now; +yield goods; pupuk roll | xpFeed |
| collectWell | target is the sumur (WRONG_TARGET); full (NOT_READY, detail.ms) | +wellWater air; collectedAt = now | 0 |
| sell | qty integer ≥ 1 (INVALID_QTY); enough goods (NOT_ENOUGH_GOODS) | −qty goods; +price × qty koin | 0 |
| mill | millable (NOT_MILLABLE); qty (INVALID_QTY); goods (NOT_ENOUGH_GOODS); room (FEED_FULL, detail.max) | −qty goods; +qty × millFeed pakan | 0 |
| move | in bounds (OUT_OF_BOUNDS); free ignoring itself (OCCUPIED) | x, y updated | 0 |
| remove | not a building (NOT_REMOVABLE) | object deleted; refund for trees and animals | 0 |
| claimAid | §4.8 conditions (AID_UNAVAILABLE) | +aidCoins; aidClaimedAt = now | 0 |

#### §4.11 Invariants (`checkInvariants`; also enforced by the save validator)

- `coins`, `xp`, `feed`, `fertilizer`, `bottles`, `water.amount` and every goods count are integers
  ≥ 0; goods entries are ≥ 1 and ≤ `GOOD_MAX`; `coins ≤ COINS_MAX`; `xp ≤ XP_MAX`.
- `water.amount ≤ waterHardCap`, `feed ≤ feedCap`, `fertilizer ≤ fertilizerCap`, `bottles ≤ bottleCap`.
- Every object key equals its `id`, matches `OBJECT_ID_RE` and its number is below `nextId`; at most
  `MAX_OBJECTS` objects; every `defId` and goods key is a known catalog id.
- Footprints are in bounds and do not overlap. Exactly one building of each `BuildingId`.
- Trees: `harvests` is an integer ≥ 0.
- Animals: `0 ≤ bottlesTaken ≤ bottlesToAdult`; `adult` is true exactly when
  `bottlesTaken === bottlesToAdult`; a baby has `fedAt === null`.
- Buildings: `collectedAt` is null unless the building is the sumur.
- Every timestamp is a finite integer in `[TIME_MIN = 2020-01-01, TIME_MAX = 2100-01-01]`; `rng` is an
  integer in `[0, 2^32)`.

### §5 Data tables (all values live in `src/core/data/catalog.ts`)

`catalogTypes.ts` defines `Catalog` with `economy` (§5.1), `levels` (§5.2: `{ level, xp, rewardCoins,
rewardWater }`), `crops` (§5.3), `trees` (§5.4), `animals` (§5.5), `goods` (§5.6), `buildings` (§5.7)
and `startingPlots` (§5.8). Records are keyed by id, every definition has `id` and `name` (display
name, Indonesian). Durations are in ms; "m" = minutes, "j" = jam (hours).

#### §5.1 Economy

| Key | Value |
|---|---|
| startCoins | 150 |
| startWater / waterRegenCap / waterHardCap | 30 / 30 / 999 |
| waterRegenMs | 180000 (3 m) |
| wellWater / wellRefillMs | 10 / 14400000 (4 j) |
| startFeed / feedCap | 0 / 50 |
| startFertilizer / fertilizerCap | 2 / 99 |
| startBottles / bottleCap / bottlePrice | 0 / 999 / 20 |
| plotBase / plotPerLevel / plotMax | 12 / 4 / 80 |
| reviveMultiplier | 2 |
| removeRefundPercent | 20 |
| xpWaterPlot / xpFertilize / xpBottle | 1 / 1 / 1 |
| aidCoins / aidCooldownMs | 50 / 600000 (10 m) |

Constants in `constants.ts`: `FARM_SIZE = 24`, `MAX_OBJECTS = 300`, `COINS_MAX = 1_000_000_000`,
`XP_MAX = 1_000_000_000`, `GOOD_MAX = 999_999`, `SAVE_VERSION = 1`, `TIME_MIN`, `TIME_MAX`.

#### §5.2 Levels (cumulative XP; unlocks are derived from each definition's `unlockLevel`)

| Level | XP | Reward koin | Reward air | Unlocks |
|---|---|---|---|---|
| 1 | 0 | 0 | 0 | tomat |
| 2 | 20 | 100 | 30 | padi, jeruk |
| 3 | 80 | 200 | 30 | cabai, mangga, ayam |
| 4 | 180 | 300 | 30 | jagung |
| 5 | 330 | 650 | 20 | singkong, rambutan, kambing |
| 6 | 580 | 750 | 10 | bawang_merah |
| 7 | 980 | 850 | 10 | kelapa |
| 8 | 1580 | 950 | 10 | sapi |
| 9 | 2430 | 1050 | 10 | – |
| 10 | 3530 | 1150 | 10 | durian |
| 11 | 4930 | 1250 | 10 | – |
| 12 | 6630 | 1400 | 10 | – |
| 13 | 8630 | 1550 | 10 | – |
| 14 | 11030 | 1700 | 10 | – |
| 15 | 13830 | 1900 | 10 | – |
| 16 | 17030 | 2000 | 15 | – |
| 17 | 20730 | 2100 | 15 | – |
| 18 | 24930 | 2200 | 15 | – |
| 19 | 29730 | 2300 | 15 | – |
| 20 | 35130 | 2400 | 15 | – |

#### §5.3 Crops (each uses 1 air per watering; `yield` 1)

| id | name | unlockLevel | seedCost | growMs | witherMs | product | xpHarvest |
|---|---|---|---|---|---|---|---|
| tomat | Tomat | 1 | 10 | 60000 (1 m) | 7200000 (2 j) | tomat | 1 |
| padi | Padi | 2 | 10 | 14400000 (4 j) | 28800000 (8 j) | gabah | 2 |
| cabai | Cabai | 3 | 15 | 120000 (2 m) | 7200000 (2 j) | cabai | 1 |
| jagung | Jagung | 4 | 15 | 28800000 (8 j) | 43200000 (12 j) | jagung | 4 |
| singkong | Singkong | 5 | 18 | 86400000 (24 j) | 86400000 (24 j) | singkong | 13 |
| bawang_merah | Bawang Merah | 6 | 22 | 43200000 (12 j) | 64800000 (18 j) | bawang_merah | 7 |

#### §5.4 Trees (2×2 footprint)

| id | name | unlockLevel | cost | waterPerCycle | cycleMs | fruit | fruitYield | xpWater | xpHarvest |
|---|---|---|---|---|---|---|---|---|---|
| jeruk | Jeruk | 2 | 260 | 3 | 43200000 (12 j) | jeruk | 6 | 1 | 4 |
| mangga | Mangga | 3 | 320 | 2 | 14400000 (4 j) | mangga | 4 | 1 | 2 |
| rambutan | Rambutan | 5 | 480 | 3 | 28800000 (8 j) | rambutan | 6 | 1 | 5 |
| kelapa | Kelapa | 7 | 620 | 3 | 7200000 (2 j) | kelapa | 4 | 1 | 4 |
| durian | Durian | 10 | 1800 | 4 | 43200000 (12 j) | durian | 4 | 1 | 10 |

#### §5.5 Animals

| id | name / babyName | unlockLevel | cost | size | bottlesToAdult | bottleIntervalMs | feedPerMeal | cooldownMs | product × yield | xpFeed | fertilizerChance | maxOwned |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ayam | Ayam / Anak Ayam | 3 | 600 | 1×1 | 2 | 300000 (5 m) | 3 | 300000 (5 m) | telur × 1 | 2 | 0.2 | 10 |
| kambing | Kambing / Cempe | 5 | 2000 | 2×2 | 3 | 900000 (15 m) | 4 | 900000 (15 m) | susu_kambing × 1 | 3 | 0.3 | 6 |
| sapi | Sapi / Pedet | 8 | 6000 | 2×2 | 6 | 3600000 (1 j) | 8 | 10800000 (3 j) | susu_sapi × 2 | 7 | 0.5 | 4 |

#### §5.6 Goods (sell price in koin; `millFeed` = pakan per item, absent = cannot be milled)

| id | name | price | millFeed |
|---|---|---|---|
| tomat | Tomat | 12 | 1 |
| gabah | Gabah | 14 | 2 |
| cabai | Cabai | 17 | 2 |
| jagung | Jagung | 20 | 4 |
| singkong | Singkong | 24 | 10 |
| bawang_merah | Bawang Merah | 30 | 6 |
| jeruk | Jeruk | 17 | 3 |
| mangga | Mangga | 18 | 2 |
| rambutan | Rambutan | 24 | 4 |
| kelapa | Kelapa | 32 | 3 |
| durian | Durian | 98 | 5 |
| telur | Telur | 60 | – |
| susu_kambing | Susu Kambing | 85 | – |
| susu_sapi | Susu Sapi | 85 | – |

#### §5.7 Buildings (movable, never removable)

| id | name | start (x, y) | size | Tap action |
|---|---|---|---|---|
| rumah | Rumah | (1, 1) | 3×3 | object info (decorative farmhouse) |
| gudang | Gudang | (5, 1) | 3×3 | Gudang panel (inventory and selling) |
| penggilingan | Penggilingan Pakan | (9, 1) | 3×3 | mill panel |
| sumur | Sumur | (13, 1) | 2×2 | collect water when full, otherwise well panel |

#### §5.8 Starting farm (`newGame(now, catalog, seed = 1)`, the FarmVille 2 tutorial farm)

- `coins 150`, `xp 0`, `water { amount: 30, updatedAt: now }`, `feed 0`, `fertilizer 2`,
  `bottles 0`, `goods {}`, `rng = seed >>> 0`, `aidClaimedAt null`, `createdAt = lastSeenAt = now`.
- The four buildings at their §5.7 positions, ids allocated in the order rumah, gudang, penggilingan,
  sumur; the sumur starts full (`collectedAt null`).
- `startingPlots`: nine plots on tiles x 10–12, y 10–12, allocated in row-major order (y, then x).
  The first four hold a ripe Tomat (`plantedAt = wateredAt = now − growMs`), the next four a seeded
  Tomat (`plantedAt = now`, `wateredAt = null`), the last one is empty.

#### §5.9 Catalog validation (`validateCatalog`)

- Ids are unique and equal their keys; names are non-empty.
- `unlockLevel` in [1, 20]; costs, prices, yields and caps are positive integers; durations are
  positive integer multiples of 1000; `fertilizerChance` in [0, 1].
- Every product and fruit is a known good; `millFeed`, when present, is an integer ≥ 1.
- Every crop's product sells for more than its seed cost; `waterPerCycle ≤ waterRegenCap`;
  `feedPerMeal ≤ feedCap`; at least one level-1 crop has `seedCost ≤ startCoins`.
- The level table has 20 rows, starts at XP 0 and strictly increases; rewards are ≥ 0.
- Buildings and starting plots are in bounds and do not overlap.

### §6 Rendering

#### §6.1 Renderer

- `WebGLRenderer({ canvas, antialias: !e2e && devicePixelRatio < 2, powerPreference: 'high-performance' })`.
- `setPixelRatio(min(devicePixelRatio, 2))` (1 under e2e); size with a `ResizeObserver` and
  `setSize(w, h, false)`.
- Default `outputColorSpace` (sRGB), `NoToneMapping`, shadow maps off, `scene.background = #BFE9FF`.
- Never set `failIfMajorPerformanceCaveat`. Construction is wrapped in try/catch and falls back to the
  §6.10 message.

#### §6.2 Camera (`IsoCameraRig`)

- `OrthographicCamera`, yaw 45°, pitch `atan(1/√2)` (35.264°). Position `target + (0.577, 0.577,
  0.577) × 60`, looking at the target; near 0.1, far 200.
- `VIEW_SIZE = 14` world units across the shorter canvas side at zoom 1: `half = 7`; landscape
  `top = half`, `right = half × aspect`; portrait `right = half`, `top = half / aspect`.
- Zoom range [0.4, 2.5], default 1. The target is clamped to x, z in [−2, 26]; the initial target is
  (11.5, 0, 11.5), the centre of the starting plots.
- Pixels per world unit: `ppu = shortSidePx / VIEW_SIZE × zoom`. The camera never rotates.

#### §6.3 Coordinates

- An object's world position is its footprint centre `(x + w/2, 0, y + h/2)`.
- World to grid: `floor(worldX)`, `floor(worldZ)`.
- Screen coordinates are always CSS px relative to the viewport (`canvas.getBoundingClientRect()`),
  never drawing-buffer px.

#### §6.4 Input

- Pointer Events only, plus `wheel` with `passive: false`; never add touch or mouse listeners.
  The canvas has `touch-action: none`; use pointer capture on down; prevent `contextmenu` and
  Safari's `gesturestart`.
- `render/input/gestures.ts` is a pure recognizer (no DOM). Input: pointer down/move/up/cancel records.
  Output: `tap`, `panStart`/`pan`/`panEnd`, `pinch` (scale, anchor), `wheel`,
  `paintStart`/`paint`/`paintEnd`/`paintCancel`.
- Tap: up within the slop (4 px mouse, 10 px touch) and within 600 ms, with no second pointer during
  the gesture. Mouse taps use the left button only.
- Paint: when the primary pointer goes down (left button or a single touch) the recognizer calls
  `canPaintAt(point)`, injected by `app/interaction`. If it returns true, moving beyond the slop starts
  a paint stroke (`paintStart`, `paint` for each move, `paintEnd` on up); otherwise the drag pans.
- Pan: "grab the ground" (ray–plane intersection before and after). Right or middle mouse drag always
  pans.
- Pinch: two pointers; zoom by the distance ratio anchored at the midpoint. A second pointer cancels an
  active paint stroke (`paintCancel`). Lifting one finger after a pinch continues as a pan and never
  produces a tap or paint.
- Wheel and ctrl+wheel: zoom factor `exp(−deltaY × 0.0015)` anchored at the cursor.
- Hover only for `pointerType === 'mouse'`.
- Keyboard: Esc cancels the mode or closes the panel; `+`/`−` zoom; arrows and WASD pan.

#### §6.5 Picking (in this order)

1. Bubbles: within `0.6 × 44 px` of a bubble's screen centre.
2. `Raycaster.setFromCamera` against object meshes (`userData.objectId`) and invisible animal proxy
   boxes (footprint × 0.8 height); the nearest hit wins. Call `scene.updateMatrixWorld()` first.
3. The ground plane y = 0, giving a tile inside the farm (and the object on that tile, if any).
4. Otherwise none.

Raycasting what is visible means clicking a tree's canopy selects the tree. Ghost meshes, overlays and
scenery are never pickable.

#### §6.6 Model kit and cache

```ts
type Vec3 = readonly [number, number, number];
interface Xf { p?: Vec3; r?: Vec3; s?: Vec3 | number }
class ModelBuilder {
  box(w, h, d, color, xf?); cyl(rTop, rBot, h, seg, color, xf?); cone(r, h, seg, color, xf?);
  ico(r, detail: 0 | 1, color, xf?); wedge(w, h, d, color, xf?); include(b: ModelBuilder, xf?);
  build(meta: { footprint: { w: number; h: number } }): ModelData;
}
interface ModelData { geometry: BufferGeometry; tris: number; height: number; anchor: Vector3; headZ?: number }
```

- Each part: `toNonIndexed()`, delete `uv`, add a Float32 `color` attribute from `new Color(hex)`
  (which converts sRGB to linear), apply the transform.
- Merge with `mergeGeometries(parts)` and throw if the result is null; then `computeVertexNormals()`,
  compute the bounding box and sphere, and dispose the parts.
- Local frame: origin at the footprint centre on the ground; +z is the model's front.
- Cache keys: `plot:empty`, `plot:seeded`, `plot:<crop>:1` … `plot:<crop>:4` (crop on the wet bed),
  `plot:withered`, `fx:fertilized`, `tree:<id>:0` … `tree:<id>:3`, `tree:<id>:fruit-unripe`,
  `tree:<id>:fruit-ripe`, `animal:<id>:baby`, `animal:<id>:adult`, `building:rumah`,
  `building:gudang`, `building:penggilingan`, `building:sumur:full`, `building:sumur:empty`,
  `deco:<type>:<variant>`.
- `ModelLibrary.get(key)` builds lazily and caches. Geometries are shared and are never disposed when
  an object is removed; `disposeAll()` runs only on teardown.
- An object's group holds its base mesh plus optional overlay meshes (tree fruit, `fx:fertilized`).
- Shared materials (`materials.ts`):

| Name | Definition |
|---|---|
| solid | `MeshLambertMaterial({ vertexColors: true, flatShading: true })` |
| ghost | solid + `transparent`, opacity 0.55, `depthWrite: false`, tinted green #4CAF50 or red #E53935 |
| shadow | `MeshBasicMaterial` black, opacity 0.22, `depthWrite: false`, polygonOffset −1/−1 |
| overlay | basic material for highlight and footprint quads |
| bubbles | one `MeshBasicMaterial` per icon (atlas clone, `transparent`, `depthTest: false`) |

#### §6.7 Layers, heights and render order

| Layer | y | renderOrder |
|---|---|---|
| ground | 0 | 0 |
| grid overlay (`LineSegments`, white, opacity 0.25; place, move and plow modes only) | 0.005 | 2 |
| blob shadows (one `InstancedMesh` of a 16-segment circle, radius 0.45 × footprint) | 0.012 | 1 |
| highlight and footprint quads | 0.02 | 3 |
| plot soil top | 0.08 | 0 |
| ghost | – | 4 |
| bubble backgrounds (one `InstancedMesh`) | – | 10 |
| bubble icons (one `InstancedMesh` per icon) | – | 11 |

- Never place two coplanar surfaces at the same height.
- Bubble quads: the geometry is pre-rotated by the camera's fixed quaternion; scale `44 / ppu`,
  updated on zoom; `frustumCulled = false`; capacity doubles when it runs out.

#### §6.8 Lighting

- `HemisphereLight(#FFFFFF, #6B8E4E, 1.6)` plus `DirectionalLight(#FFF4E0, 2.2)` at target +
  (−6, 12, 8). Tune in the gallery. No shadow maps; blob shadows only.

#### §6.9 Animation (Sprint 14)

- Pop: scale 0.7 → 1 with `outBack` over 250 ms when an object is added or its model key changes.
- Harvest squash: 1 → 1.15 → 1 over 150 ms.
- Harvest fly: a DOM icon of the good arcs from the object's screen point to the Gudang toolbar button
  over 600 ms.
- Bubble bob ±3 px at 1.2 Hz; fertilized sparkle twinkle (scale 0.8–1.2 at 1 Hz); pan inertia.
- Animal wander: a seeded state machine (`mulberry32(hashString(id))`): idle 1–4 s, then walk at
  0.3–0.6 units/s to a random point inside the footprint inset by 0.2 (0.25 for 1×1); heading is lerped,
  the body bobs while walking, chickens sometimes dip to peck.
- All animation is off under `?e2e=1` (unless `anim=1`), under `prefers-reduced-motion`, and with
  the "Kurangi Animasi" setting.

#### §6.10 Loop and lifecycle

- Render on demand (`requestRender()`). Render continuously only while a gesture, inertia, tween or
  wander is active.
- When `document.hidden`: stop the loop and the 1 Hz tick and flush the save. When visible again:
  `store.tick()`, resync, render.
- `webglcontextlost`: `preventDefault`, stop, show the overlay "Memulihkan grafis…". On
  `webglcontextrestored`: set the atlas `needsUpdate`, resume, hide the overlay.
- If WebGL2 is missing or the renderer throws, `#fatal` shows: "Browser ini tidak mendukung WebGL2.
  Aktifkan akselerasi grafis atau gunakan Chrome, Firefox, atau Safari terbaru."

#### §6.11 Performance budgets

- Triangles per model: plot + crop 500; tree 900; fruit overlay 300; `fx:fertilized` 100; animal 600;
  building 2000; decoration 300; merged scenery 40k.
- Draw calls ≤ 2 × objects + 20.
- `stress` fixture: ≤ 320 draw calls, ≤ 220k triangles, `profileSync()` ≤ 8 ms.
- Memory: geometries ≤ (model keys in use) + 15, textures ≤ 4, both stable over 50 reset cycles.
- Production JS ≤ 1.3 MB minified.
- Real hardware (checked by a human with `?perf=1`, never asserted on SwiftShader): 60 fps for the
  stress scene on a desktop integrated GPU at 1080p, ≥ 45 fps with 150 objects on a mid-range phone.

### §7 Art direction

#### §7.1 Style

- Chunky low-poly primitives, flat shading, bright saturated colours. Cylinders and cones use 5–8
  segments; icosahedra use detail 0–1.
- 1 world unit = 1 tile. Models face +z, stand at y = 0 and are centred on their footprint.
- Small hash-seeded variation inside a model is allowed; every model is deterministic per key.

#### §7.2 Palette (sRGB hex)

| Group | Colours |
|---|---|
| Sky and ground | sky #BFE9FF; grass #8CC63F; grassAlt #7FB935; grassOuter #6FA832 |
| Soil | soilDry #C9965F; soilWet #8A5A3A; furrow #6E4630; seed #4E342E |
| Plants | leaf #4CAF50; leafDark #2E7D32; leafLight #8BD45A; stem #6B8E23; trunk #8B5A2B; trunkDark #6D4C41 |
| Crop produce | padiGold #E8C547; cornYellow #FFD54F; cornHusk #A5D66B; tassel #B08D57; chiliRed #E53935; tomatoRed #F44336; unripeGreen #9CCC65; shallot #A23E5C; cassavaSkin #8D6E63; cassavaStem #A1887F; flowerWhite #FFFDE7 |
| Wither and fertilizer | wither #8D6E4A; witherDark #6D5A3F; sparkle #FFE082; sparkleCore #FFF8E1 |
| Fruit | mango #FFB300; mangoBlush #FF7043; rambutan #D32F2F; durian #9E9D24; durianSpike #6B6B1F; coconut #7CB342; coconutBrown #795548; frond #43A047; orange #FB8C00 |
| Animals | white #FAFAFA; black #37474F; comb #E53935; beak #FFB300; chick #FFE082; pink #F8BBD0; horn #FFF3E0; goat #D2B48C; goatDark #8D6E63 |
| Buildings and props | woodLight #C19A6B; woodDark #8D6E63; barnRed #C0392B; roof #7B2E2E; trim #FFFFFF; stone #9E9E9E; stoneDark #757575; water #4FC3F7; metal #B0BEC5; sack #E6D3A3; path #D7B98E; houseWall #F5E6C8; houseRoof #D9534F; window #81D4FA; chimney #9E9E9E |
| Flowers | petalPink #F48FB1; petalYellow #FFEE58; petalPurple #BA68C8 |

#### §7.3 Model specs (heights in world units)

- **Plot.** Raised bed 0.92 × 0.08 × 0.92 with three furrow ridges (0.86 × 0.03 × 0.14, `furrow`).
  `plot:empty` is dry soil; `plot:seeded` is dry soil with 9 seed dots; crop stages 1–4 sit on wet
  soil; `plot:withered` is dry soil with 6–9 stalks drooping 40–60° in `wither`/`witherDark`.
- **`fx:fertilized`.** 7 small gold octahedra (radius 0.03–0.05, `sparkle` and `sparkleCore`) at
  heights 0.12–0.3 over a 0.8 × 0.8 area; trees use it scaled ×2 around the trunk base.
- **Crop layout.** A 3×3 grid at ±0.28 for padi and bawang merah; a 2×2 grid at ±0.22 for the others.

| Crop | Stage 1 | Stage 2 | Stage 3 | Stage 4 (ripe) |
|---|---|---|---|---|
| padi (clumps of 4–5 three-segment blade cones, tilted ±12°) | h 0.10, leafLight | h 0.22 | h 0.38, pale panicles | h 0.45; padiGold upper blades and drooping golden panicles |
| jagung | 2-leaf sprout, h 0.12 | stalk h 0.35 with 4 angled flat leaves | h 0.7, tassel bud | h 0.95; tassel cones; 1–2 cornYellow cobs partly wrapped in cornHusk |
| cabai | sprout | leafDark bush h 0.18 | h 0.30, flowerWhite dots | h 0.38; 5–6 hanging chiliRed cones and 1–2 green ones |
| tomat (woodLight stake from stage 2) | sprout | bush h 0.22 | h 0.40, unripeGreen balls | h 0.48; 4–5 tomatoRed balls |
| bawang merah (tufts of 4–6 thin four-segment tubes) | h 0.08 | h 0.16 | h 0.24 | h 0.26 with yellowing tips; shallot bulbs half-exposed at soil level |
| singkong (cassavaStem stems topped by palmate clusters of 5–7 flat cones) | h 0.12 | h 0.35 | h 0.60 | h 0.70; 2–3 cassavaSkin tubers lying on the soil |

- **Trees** (2×2 footprint, canopy diameter ≤ 1.9). Stages 0–2 have heights 0.4 / 1.0 / 1.7; stage 0
  is a sapling (thin stem, 2–4 leaves, dark soil mound). `fruit-unripe` uses the same positions as
  `fruit-ripe` at 70% size in `unripeGreen` (kelapa: small `coconut` green).

| Tree | Stage 3 height | Shape | Ripe fruit |
|---|---|---|---|
| jeruk | 2.0 | compact round leafLight canopy | about 10 small `orange` balls |
| mangga | 2.6 | broad dome of 3–4 leafDark icosahedra | 8 oval `mango` fruits with `mangoBlush` |
| rambutan | 2.3 | dense round canopy | 6 clusters of 3–4 small `rambutan` balls with tiny spikes |
| kelapa | 3.2 | palm: curved trunk of 6 offset tapered segments in alternating tones; 7–8 drooping `frond` strips | 4–5 coconuts, mixed `coconut` and `coconutBrown` |
| durian | 3.0 | tall trunk, tiered narrow canopy | 4 large spiky `durian` fruits (spikes `durianSpike`) on low branches |

- **Animals**

| Animal | Adult | Baby |
|---|---|---|
| ayam (1×1) | h 0.45; white egg-shaped body; comb, wattle, beak; tail feathers; orange legs | Anak Ayam: yellow (`chick`) round body, h 0.2 |
| kambing (2×2) | h 0.75, length 0.8; `goat` box body; head with backward horns and a `goatDark` beard; four legs; upright tail | Cempe: 60% size, larger head, no horns |
| sapi (2×2) | h 1.0, length 1.2; white body with black patches that protrude 0.01 (no coplanar faces); pink muzzle and udder; `horn` horns; tufted tail | Pedet: 60% size, larger head, no horns or udder |

- **Buildings**

| Building | Description |
|---|---|
| rumah (3×3) | `houseWall` walls 2.4 × 1.4 × 2.0; `houseRoof` gable roof with 0.15 overhang; door; two `window` windows with `trim` frames; `chimney`; small porch with two posts; height ≤ 2.6 |
| gudang (3×3) | `barnRed` barn body 2.6 × 1.8 × 2.4; `roof` gable; `trim` trims; front double doors with an X; loft window |
| penggilingan (3×3) | `metal` silo (r 0.6, h 2.6) with a cone roof; small wooden shed; yellow hopper; `sack` sacks |
| sumur (2×2) | `stone` ring (r 0.45, h 0.45); two posts h 1.3; crank; small red gable roof; bucket. `full`: `water` disc at y 0.35; `empty`: `stoneDark` disc at y 0.1 |

- **Decorations:** pagar (fence: straight and corner), semak (bush ×3), batu (rock ×3), bunga
  (flowers ×3), border trees ×3 (broadleaf and conifer, no fruit), jalan (path stones).

#### §7.4 Scenery

- A fence runs along the farm boundary. An 8-tile ring outside it holds seeded decoration placements on
  `grassOuter`, merged into one geometry with one draw call. Nothing is placed inside the farm.

#### §7.5 Icons

- IconIds: `air`, `pakan`, `pupuk`, `botol`, `koin`, `xp`, `gembok` (lock), `jam` (clock), `layu`
  (wilted leaf), `ayam`, `kambing`, `sapi`, plus all 14 GoodIds (26 icons).
- Drawn with Canvas2D (paths, arcs, gradients) with a dark outline (#3E2723, about 6% of the size).
  No text and no emoji.
- Atlas of 6 × 5 cells of 128 px. The UI calls `drawIcon(ctx, id, size)` on small `<canvas>` elements.

#### §7.6 Visual review checklist (the tester checks screenshots)

1. Each crop is distinguishable at stage 4, and stages visibly grow from 1 to 4.
2. Empty and seeded (dry) soil look different from watered (wet) soil.
3. Withered plots read as dead and brown; fertilized plots and trees show visible gold sparkles.
4. Tree species and their fruit are distinguishable; the palm reads as a palm; unripe (green) and ripe
   fruit are distinguishable.
5. Animal species and baby versus adult are distinguishable.
6. Buildings are recognizable and consistently scaled; the sumur shows full versus empty.
7. Icons are legible at 32 px and distinct from each other.
8. Faceted flat look; no missing faces, z-fighting or flicker.
9. Bubbles are legible at 44 px and keep the same size at every zoom; all UI text is Indonesian.

### §8 UI/UX

#### §8.1 Layout

- HUD top-left: level badge and XP bar ("Lv 3 · 45/100 XP": XP inside the level / XP needed for the
  next level; "MAX" at level 20).
- HUD top-right: koin; air "12/30" with a "+1 02:13" countdown below the cap; pakan "8/50"; pupuk;
  botol.
- Toolbar at the bottom (full width on mobile, scrollable when narrow): Toko, Gudang, Bajak, Pupuk,
  Pindah, Jual, Pengaturan. The active tool is highlighted.
- Mode chip at the top centre whenever the mode is not normal: "Mode Tanam: Tomat", "Mode Bajak",
  "Mode Pupuk", "Mode Pindah", "Mode Jual", "Pasang: Jeruk", with a "Batal" button.
- Panels: one at a time; on desktop a centred modal up to 520 px wide; at ≤ 600 px a bottom sheet up
  to 70dvh, scrollable. Close with "Tutup", Esc or the backdrop.
- Respect safe-area insets; use `100dvh`; touch targets ≥ 44 px; font stack
  `system-ui, "Segoe UI", Roboto, "Noto Sans", sans-serif`.

#### §8.2 Tapping objects in normal mode

| Target | Result |
|---|---|
| Empty plot | Seed Picker; choosing a crop plants this plot and enters seed mode with that crop |
| Seeded plot | water |
| Growing plot | object info |
| Ripe plot | harvest |
| Withered plot | Withered dialog: "Bersihkan" (clearPlot) or "Pulihkan (N koin)" (revive) |
| Tree needing water | water |
| Tree growing | object info |
| Tree ready | harvest |
| Hungry baby | feed (Botol Susu) |
| Hungry adult | feed (pakan) |
| Animal in cooldown | object info |
| Sumur full | collectWell |
| Sumur refilling | well panel |
| Gudang | Gudang panel |
| Penggilingan | mill panel |
| Rumah | object info |
| Empty tile | close the panel and deselect |

Desktop only: hovering an object shows a tooltip with its name, status text and time left, refreshed at
1 Hz.

#### §8.3 Modes and drag-to-act

- **seed** (from the Seed Picker or the Toko's Benih tab): tapping or dragging over empty plots plants
  the active crop; other targets behave as in normal mode. Exits with "Batal", Esc, or when koin run out
  (toast).
- **plow** (Bajak): a plot ghost follows the hover (mouse) or the last tap (touch); tapping or
  dragging over free tiles plows. It stays active after `LIMIT_REACHED` (toast) until "Batal" or Esc.
- **fertilize** (Pupuk): tapping or dragging over plots and trees fertilizes them; exits when pupuk
  reaches 0 (toast), with "Batal" or Esc.
- **move** (Pindah, or Pindahkan in object info): the first tap picks any object (buildings
  included); its ghost follows; tapping a destination moves it; Esc or "Batal" cancels.
- **sell** (Jual): tapping an object opens a confirmation ("Jual Jeruk? Kamu mendapat 52 koin." or,
  for plots, "Hapus petak ini? Tanaman di atasnya akan hilang."); buildings show the `NOT_REMOVABLE`
  toast.
- **place** (Toko → Pohon or Hewan): ghost plus footprint quad, green when the spot is valid and
  affordable, otherwise red with the reason. Mouse: the ghost follows the hover and a click places.
  Touch: a tap moves the ghost and "Pasang" (or tapping the same tile again) places. One item per
  purchase; the mode ends after placing; Esc or "Batal" cancels without cost.
- **Drag-to-act** ("drag massal", FarmVille 2). A stroke starts only when the pointer goes down on a
  valid target for the mode's paint action:

| Mode | Paint action | Valid targets |
|---|---|---|
| normal | water | seeded plots (the stroke started on a seeded plot) |
| normal | harvest | ripe plots (the stroke started on a ripe plot) |
| seed | plant | empty plots |
| plow | plow | free tiles |
| fertilize | fertilize | plots and trees that can be fertilized |

  Each target is acted on at most once per stroke; the camera does not move during a stroke; other
  targets are skipped silently; the first failure for a missing resource (koin, air, pupuk) ends the
  stroke with one toast. Right/middle mouse drags and two-finger touches always pan and zoom.

#### §8.4 Panels and modals

- **Seed Picker** (`seed`): every crop with icon, cost, grow time and product; locked crops are
  disabled and show "Level N".
- **Toko** (`shop`): tabs Benih (crops → seed mode), Pohon and Hewan (→ place mode), Perlengkapan
  (Botol Susu ×1, ×5, ×10). Each item shows its icon, name, price, key stats, lock level and
  affordability.
- **Gudang** (`storage`): goods with icon, count and price; quantity 1 / 5 / Semua; "Jual" with the
  total; a summary of pakan, pupuk and botol.
- **Penggilingan** (`mill`): millable goods the player owns with their ratio ("1 Jagung → 4 Pakan");
  quantity 1 / 5 / Maks; "Giling"; a pakan bar "x/50".
- **Sumur** (`well`): status ("Penuh" or the refill time), "Ambil Air (+10)", and a line about natural
  regeneration ("+1 air setiap 3 menit sampai 30").
- **Object info** (`objectInfo`): name (baby names for babies), status text, time left, progress bar,
  a fertilized badge, and the actions Pindahkan, Jual or Hapus (not for buildings), and Beri Pupuk
  (when allowed).
- **Pengaturan** (`settings`): Suara (on/off), Kurangi Animasi, Mulai Ulang (with confirmation), the
  version.
- Modals: confirm, level-up (queued), withered.

#### §8.5 Feedback

- Floating text near the object, from events: "+1 Tomat", "+1 XP", "−1 Air", "−10 koin",
  "+1 Pupuk!", "+10 Air". It rises 40 px over 1.2 s; under e2e it is static for 2 s.
- Toasts: at most 3 at a time, 3 s each, `aria-live`.
- Level-up modal: "Naik Level N!", "+X koin", "+Y air", the unlocked items, and the line "Semua
  tanaman yang sudah disiram langsung siap panen!" when `levelBonus` ripened or revived anything.
  Several level-ups queue.
- Aid banner: "Bantuan Desa" with the button "Ambil 50 koin" while aid is available.
- Banners (Sprint 16): "Progres tidak dapat disimpan" when storage is unavailable; a read-only banner
  for a save from a newer version.
- Onboarding hints (Sprint 16), the FarmVille 2 tutorial order: "Panen 4 tomat yang sudah matang.
  Klik tomatnya!" → "Siram 4 tomat yang baru ditanam." → "Tanam benih di petak yang kosong." → "Jual
  tomat di Gudang untuk mendapat koin." → "Pilih Bajak, lalu klik rumput untuk membuat petak baru." →
  "Hebat! Selamat bertani!"

#### §8.6 Strings, glossary and status texts

- All UI text lives in `src/ui/strings.id.ts`; display names live in the catalog.
- Vocabulary: petak, lahan, benih, bibit, siram, panen, layu, bersihkan, pulihkan, pupuk, bajak,
  gudang, sumur, air, pakan, penggilingan pakan, botol susu, koin, XP, level, tumbuh, siap panen,
  lapar, kenyang, beri pakan, beri susu, ambil air, jual, giling, pindahkan, hapus, pasang, batal,
  tutup, toko, dewasa, anak.
- Durations: "45d", "4m 05d", "1j 05m", "1h 02j" (h = hari, from 24 hours). Numbers use
  `Intl.NumberFormat('id-ID')`, so 1250 shows as "1.250".
- Status texts (keys used by `objectInfo` and the tooltip):

| Key | Text |
|---|---|
| plot.empty | "Petak kosong" |
| plot.seeded | "Perlu disiram" |
| plot.growing | "Tumbuh · siap dalam {time}" |
| plot.ripe | "Siap panen · layu dalam {time}" |
| plot.withered | "Layu" |
| tree.needsWater | "Perlu disiram ({n} air)" |
| tree.growing | "Berbuah dalam {time}" |
| tree.ready | "Siap panen" |
| baby.hungry | "Minta susu ({taken}/{need} botol)" |
| baby.cooldown | "Kenyang · minta susu lagi dalam {time}" |
| adult.hungry | "Lapar · butuh {n} pakan" |
| adult.cooldown | "Kenyang · lapar lagi dalam {time}" |
| sumur.full | "Penuh · ambil 10 air" |
| sumur.refilling | "Terisi penuh dalam {time}" |
| building.idle | "Bangunan" |

#### §8.7 Error messages

| Code | Message |
|---|---|
| NOT_FOUND | "Objek tidak ditemukan." |
| WRONG_TARGET | "Aksi ini tidak bisa dilakukan di sini." |
| OUT_OF_BOUNDS | "Tidak bisa ditempatkan di luar lahan." |
| OCCUPIED | "Tempat ini sudah terisi." |
| LIMIT_REACHED (plots) | "Batas petak tercapai ({limit}). Naik level untuk menambah petak." |
| LIMIT_REACHED (animals) | "Jumlah {name} sudah maksimal ({limit})." |
| LIMIT_REACHED (objects, goods, coins, bottles) | "Batas tercapai." |
| NOT_ENOUGH_COINS | "Koin tidak cukup." |
| NOT_ENOUGH_WATER | "Air tidak cukup. Tunggu air terisi atau ambil dari Sumur." |
| NOT_ENOUGH_FEED | "Pakan tidak cukup. Giling hasil panen di Penggilingan Pakan." |
| NOT_ENOUGH_BOTTLES | "Botol Susu habis. Beli di Toko." |
| NOT_ENOUGH_FERTILIZER | "Pupuk habis. Beri makan hewan dewasa untuk mendapat pupuk." |
| NOT_ENOUGH_GOODS | "Barang di gudang tidak cukup." |
| FEED_FULL | "Tempat pakan sudah penuh." |
| LEVEL_LOCKED | "Terbuka di Level {level}." |
| PLOT_EMPTY | "Petak masih kosong. Tanam benih dulu." |
| PLOT_NOT_EMPTY | "Petak ini sudah ditanami." |
| ALREADY_WATERED | "Sudah disiram." |
| ALREADY_FERTILIZED | "Sudah dipupuk." |
| NOT_READY | "Belum siap. {time} lagi." |
| WITHERED | "Tanaman sudah layu." |
| NOT_WITHERED | "Tanaman ini tidak layu." |
| NOT_HUNGRY | "Hewan ini masih kenyang." |
| NOT_MILLABLE | "Barang ini tidak bisa digiling." |
| INVALID_QTY | "Jumlah tidak valid." |
| NOT_REMOVABLE | "Bangunan ini tidak bisa dijual." |
| AID_UNAVAILABLE | "Bantuan belum tersedia." |
| UNSUPPORTED | "Aksi tidak dikenal." |

#### §8.8 Test ids and accessibility

- HUD: `hud-level`, `hud-xp`, `hud-coins`, `hud-water`, `hud-water-timer`, `hud-feed`,
  `hud-fertilizer`, `hud-bottles`.
- Toolbar and mode: `btn-shop`, `btn-storage`, `btn-plow`, `btn-fertilize`, `btn-move`,
  `btn-sell-tool`, `btn-settings`, `mode-chip`, `btn-mode-cancel`.
- Panels: `panel-<PanelId>`, `btn-close`.
- Seed Picker and Toko: `seed-<cropId>`, `shop-tab-<benih|pohon|hewan|perlengkapan>`,
  `shop-item-<crop|tree|animal>-<id>`, `buy-bottles-<1|5|10>`.
- Gudang: `goods-row-<goodId>`, `sell-qty-1`, `sell-qty-5`, `sell-qty-all`, `btn-sell`.
- Penggilingan: `mill-row-<goodId>`, `mill-qty-1`, `mill-qty-5`, `mill-qty-max`, `btn-mill-go`.
- Sumur: `btn-well-collect`.
- Placement: `place-bar`, `btn-place-confirm`, `btn-place-cancel`.
- Feedback and modals: `toast`, `float-text`, `tooltip`, `modal-levelup`, `modal-confirm`,
  `btn-confirm-yes`, `btn-confirm-no`, `modal-withered`, `btn-withered-clear`, `btn-withered-revive`.
- Object info: `info-name`, `info-status`, `info-progress`, `btn-move`, `btn-remove`,
  `btn-info-fertilize`.
- Other: `aid-banner`, `btn-aid`, `banner-storage`, `overlay-tab`, `overlay-gl`, `fatal`, `hint`,
  `settings-sound`, `settings-reduced-motion`, `btn-reset`.
- Use real `<button>` elements with accessible names, visible focus, and `aria-live` for toasts.

### §9 Persistence

#### §9.1 Keys

`gaskan-bertani:save`, `gaskan-bertani:backup`, `gaskan-bertani:corrupt` (quarantine, ≤ 200 KB),
`gaskan-bertani:settings` (`{ muted, reducedMotion, hintsStep, hintsDone }`, whitelist-validated). Only
`app/persistence.ts` and `app/settings.ts` touch localStorage, always through `StorageLike` and inside
try/catch.

#### §9.2 Envelope

`{"app":"gaskan-bertani","version":1,"savedAt":<ms>,"state":GameState}`. Payloads over 1 MB are rejected
before parsing.

#### §9.3 Validation (`validateSave(raw: unknown): Result<GameState, SaveError>`)

- `SaveError`: `PARSE_ERROR`, `TOO_LARGE`, `NOT_OBJECT`, `BAD_APP`, `UNSUPPORTED_VERSION`,
  `FUTURE_VERSION`, `INVALID_SCHEMA`, `INVARIANT_VIOLATION`.
- Build a fresh object field by field from a whitelist. Never spread or `Object.assign` untrusted data.
- Check types, finiteness and integer-ness; check the id regex before using an id as a key; only known
  definition ids; then every §4.11 invariant; at most `MAX_OBJECTS` objects.

#### §9.4 Migrations

- A registry `{ [fromVersion]: (raw) => raw }`. Version 0 is a synthetic legacy format that exists only
  to exercise the pipeline: identical to v1 except `money` instead of `coins` and no `aidClaimedAt`.
- After Sprint 5, any change to the shape of `GameState` requires: bumping `SAVE_VERSION`, adding a
  migration, and adding a golden file `src/core/save/__fixtures__/vN-*.json`. Old golden files are
  never edited.

#### §9.5 Load and write

- Load order: main → backup → new game. Outcomes: `new`, `loaded`, `recoveredFromBackup`,
  `resetCorrupt`, `futureVersion`, `storageUnavailable`.
  - A corrupt main save is copied to the quarantine key.
  - A future-version save is left untouched; the game runs in memory, read-only for storage, with a
    banner.
  - If storage is unavailable, the game runs in memory with the banner "Progres tidak dapat disimpan".
- Write: debounced 400 ms after each successful dispatch, flushed on `visibilitychange → hidden` and
  `pagehide`. On the first save of a session, copy a valid existing main save to the backup key.
- Quota exceeded: remove the quarantine and retry once; if it still fails, toast "Gagal menyimpan".

#### §9.6 Tab guard (Sprint 16)

- `BroadcastChannel('gaskan-bertani')`; on boot a tab posts `hello`. An older tab that receives `hello`
  becomes inactive (no input, no saving) and shows the overlay "Gaskan Bertani sedang dibuka di tab
  lain." with the button "Main di sini", which reloads from storage and posts `hello`.
- Off under e2e unless `tabguard=1`.

### §10 Testing

#### §10.1 Layers

- Vitest (node) for core: `ManualClock`, builders, seeded property tests, scenario tests. From Sprint 4,
  `vitest.config.ts` enforces coverage thresholds for `src/core/**` when run with `--coverage`
  (lines 90%, branches 85%).
- Vitest (node) for render math and models: three's geometry, `Raycaster`, cameras and
  `BufferGeometryUtils` work without WebGL. Never construct `CanvasTexture`, `WebGLRenderer` or
  `AudioContext` in node.
- Vitest for pure UI view-models, formatting, the gesture recognizer and the interaction controller
  (with fake ports). There is no DOM test environment.
- Playwright E2E against `vite preview` of the e2e build, using the debug API, real clicks and touches,
  and screenshots.

#### §10.2 URL flags (honoured only when `__GB_DEBUG__` is true)

| Flag | Effect |
|---|---|
| `e2e=1` | frozen `ManualClock` (starts at real now); no 1 Hz tick; no animations, CSS transitions, sounds, hints or tab guard; DPR 1; antialias off; cosmetic seed 1; static floating texts |
| `fixture=<name>` | boot with that fixture (§10.6) |
| `seed=<int>` | cosmetic random seed |
| `anim=1`, `hints=1`, `tabguard=1` | turn that feature back on under e2e |
| `forceNoWebgl=1` | show the WebGL fallback |
| `perf=1` | performance overlay |
| `cat`, `key`, `zoom` | gallery only |

#### §10.3 Debug API (`window.__GB__`, debug builds only, typed in `src/debug/types.ts`)

- `ready`, `frameCount`.
- `getState()` (deep copy), `getCatalog()`, `dispatch(cmd): DispatchResult`.
- `fastForward(ms)`, `setNow(ms)` (frozen clock only), `reset(fixtureName | state)` (validated).
- `flushSave()`, `getLoadOutcome()`, `getEvents()` (ring buffer of 200).
- `tileToScreen(x, y): ScreenPoint`, `objectToScreen(id)`, `findClickPoint(id)` (samples the projected
  bounding box until `pickAt` returns that id), `screenToTile(x, y)`, `pickAt(x, y)`.
- `getCamera()`, `setCamera(partial)`, `focusTile(x, y, zoom?)`.
- `renderNow()`, `probePixel(x, y)` (renders, then `gl.readPixels` in the same task; no
  `preserveDrawingBuffer`), `getRenderInfo()` → `{ calls, triangles, geometries, textures, meshes }`.
- `getBubbles()`, `getUiState()`, `ui.open(panel, args)`, `ui.close()`, `profileSync()`.

#### §10.4 Playwright

- `e2e/gl-profiles.json`:
  ```json
  {"default": "A", "profiles": {
    "A": ["--use-angle=swiftshader", "--enable-unsafe-swiftshader", "--ignore-gpu-blocklist"],
    "B": ["--use-gl=angle", "--use-angle=swiftshader-webgl", "--enable-unsafe-swiftshader"]}}
  ```
- Playwright already adds `--no-sandbox`, `--disable-dev-shm-usage`, `--enable-unsafe-swiftshader` and
  `--headless`. Never pass `--disable-gpu`.
- `playwright.config.ts`:

```ts
import { defineConfig } from '@playwright/test';
import { readFileSync } from 'node:fs';
if (process.env.PLAYWRIGHT_BROWSERS_PATH !== '0')
  throw new Error('Run E2E via "npm run e2e" (scripts/run.mjs sets PLAYWRIGHT_BROWSERS_PATH=0 and TMPDIR).');
const gl = JSON.parse(readFileSync(new URL('./e2e/gl-profiles.json', import.meta.url), 'utf8')) as
  { default: string; profiles: Record<string, string[]> };
const args = gl.profiles[process.env.GB_GL_PROFILE ?? gl.default];
if (!args) throw new Error('Unknown GB_GL_PROFILE');
export default defineConfig({
  testDir: 'e2e', outputDir: 'test-artifacts/pw', timeout: 90_000, expect: { timeout: 15_000 },
  workers: Number(process.env.GB_PW_WORKERS ?? 1), retries: 1, forbidOnly: true,
  reporter: [['list'], ['json', { outputFile: 'test-artifacts/e2e-results.json' }]],
  use: { baseURL: 'http://127.0.0.1:4317', locale: 'id-ID', timezoneId: 'Asia/Jakarta',
    deviceScaleFactor: 1, screenshot: 'only-on-failure', trace: 'off', video: 'off',
    channel: process.env.GB_PW_CHANNEL || undefined, launchOptions: { args } },
  projects: [
    { name: 'desktop', testIgnore: /\.mobile\.spec\.ts$/, use: { viewport: { width: 1280, height: 720 } } },
    { name: 'mobile', testMatch: /\.mobile\.spec\.ts$/,
      use: { viewport: { width: 390, height: 844 }, isMobile: true, hasTouch: true } },
  ],
  webServer: { command: 'node_modules/.bin/vite preview --mode e2e', url: 'http://127.0.0.1:4317',
    reuseExistingServer: false, timeout: 60_000 },
});
```

#### §10.5 E2E conventions

- Import `test` and `expect` only from `e2e/support/test.ts`. Its fixture fails a test on any console
  `error` or `pageerror` (except an explicit allowlist) and on any request outside
  `http://127.0.0.1:4317` (`data:` and `blob:` are allowed).
- `gb.open({ fixture, flags })` loads `/?e2e=1&…`, then waits for `__GB__.ready` and
  `document.fonts.ready`.
- Never use `waitForTimeout`. Wait with `expect.poll`, `waitForFunction` or locators. To check that
  nothing renders while idle, compare `frameCount` across 60 `requestAnimationFrame` callbacks.
- Time moves only through `gb.fastForward()`, using durations from `gb.catalog()`.
- `gb.clickTile` and `gb.clickObject` call `focusTile` first and assert `visible && !occluded`. Desktop
  uses `page.mouse`; the mobile project uses `page.touchscreen.tap`. Drags use `page.mouse` moves (or
  CDP touch events on mobile). Pinch is tested with CDP `Input.dispatchTouchEvent`.
- Assert state through `gb.state()` and UI through test ids.
- At most 8 tests per file. Target ≤ 10 minutes for the whole suite on 2 vCPU.

#### §10.6 Fixtures (`src/debug/fixtures.ts`; times relative to now; every fixture passes `validateSave`)

| Fixture | Contents |
|---|---|
| fresh | `newGame` |
| starter | level 1, 150 koin, 30 air, the buildings, 4 empty plots at (10–13, 10) |
| midgame | level 10, 3000 koin, 60 air, 40 pakan, 5 pupuk, 5 botol, 2–5 of every good; plots in every stage 0–5 (some fertilized); one tree of each species in different phases (sapling needing water, first cycle growing, mature needing water, mature growing with unripe fruit, ready); a hungry baby and a hungry adult of each animal plus one adult in cooldown; sumur full |
| allStages | every VisualState laid out in rows (overview screenshots) |
| noWater | 0 air, seeded plots, sumur refilling |
| noFeed | 0 pakan, hungry adults |
| noBottles | 0 botol, hungry babies |
| withered | several withered plots, 200 koin |
| levelUp | xp one below level 2; two ripe tomatoes, two watered growing padi, one withered cabai, one seeded tomat (harvesting one tomato levels up and triggers the bonus) |
| softlock | 3 koin, no goods, only empty plots, no trees or animals |
| stress | level 20; 80 plots in all stages, 40 trees in mixed phases, 20 animals, the buildings |

#### §10.7 Screenshots

- `snap(page, 'sNN-name')` writes `test-artifacts/screens/sNN-name.png` after `renderNow()`. Names are
  stable, so later runs overwrite them.
- No pixel-golden assertions. Visual quality is reviewed by the tester against §7.6. Objective checks:
  model unit tests (palette colours, triangle counts) and `probePixel` colour probes.

#### §10.8 Verification scripts and security checklist

- `verify-bundle.mjs`: `dist/` exists; no `__GB__`, `installDebugApi` or `gallery` strings; the CSP
  meta tag is present; total JS ≤ 1.3 MB.
- `verify-hygiene.mjs`: no `.only(`, `.skip(` or `test.fixme` in `src` or `e2e`; no `innerHTML` in
  `src`; no `http(s)://` URLs in `src` except in comments.
- `doctor.mjs`: Node is in the supported range; running from the project root; the npm cache is inside
  the project; `.tmp` is writable; `Intl` id-ID formats 1250 as "1.250"; browsers are present; it
  launches Chromium with the selected GL profile and prints the WebGL2 `UNMASKED_RENDERER_WEBGL`
  string (environment variables are set before a dynamic `import('@playwright/test')`); when OS
  libraries are missing it prints the §2.1 command and exits with code 2.
- Tester security review: `verify` is green; the E2E request and console guards pass;
  `npm audit --omit=dev` is clean; the save fuzz tests exist; localStorage is used only in persistence
  and settings; no `eval`, `new Function`, `innerHTML` or `setAttribute('style')`; dev and preview
  servers bind to 127.0.0.1 only.

### §11 Conventions and Definition of Done

#### §11.1 Code

- Strict TypeScript; no `any`, no `as unknown as`, no non-null `!` in `src`.
- `import type` for types; no enums, namespaces or parameter properties.
- Files ≤ 300 lines; one model family per file. Comments in English.
- No dependencies beyond §2.4. Core returns `Result` values and throws only on programmer errors.

#### §11.2 Tests and the sprint Definition of Done

- Every task adds or updates tests. Never delete, skip or weaken a test to make it pass.
- A sprint is done when: all its tasks and Fix items are checked; `npm run check:all` is green inside
  the sandbox; from Sprint 4, `npm run test:coverage` meets the thresholds; `coder-report.md` lists
  the commands run, the tests added and the screenshots produced; decisions are appended to
  `docs/decisions.md`; nothing was written outside the project and no server is left running.

#### §11.3 Project Definition of Done (the reviewer then writes DONE.md)

- All sprints approved.
- The full §1 loop is playable through the UI in both the desktop and mobile projects: crops with
  withering, trees, animals with bottles, well, mill, fertilizer, drag-to-act, levels and the bonus.
- Every model appears in the gallery and in the game.
- The persistence, offline-progress and fuzz suites are green; the §6.11 budgets are green; the §10.8
  security checklist is clean.

### §12 Known pitfalls (read before coding)

1. `typescript@latest` (7.x) breaks typescript-eslint. Keep 6.0.3. TypeScript 6 needs explicit `types`.
2. Vite 8 uses `build.rolldownOptions`, not `rollupOptions`, and has no `esbuild` options.
3. ESLint options do not merge across config blocks; repeat the shared selectors.
4. Writes outside the project fail. Use the npm scripts and `scripts/run.mjs`; never `npx playwright`,
   global installs or sudo.
5. `npm ci` deletes the installed browsers; `pree2e` reinstalls them. Missing OS libraries mean
   BLOCKED.md (owner action).
6. Headless WebGL needs the SwiftShader opt-in profile. Never use `--disable-gpu` or
   `failIfMajorPerformanceCaveat`. SwiftShader is slow: render on demand, DPR 1, no antialias in e2e,
   no shadow maps.
7. Flaky E2E: no sleeps, a frozen clock, `fastForward`, fixtures, catalog-driven durations, the
   occlusion check before clicking, 1 worker. A test that passes only on retry becomes a Fix item.
8. Vertex colours must go through `THREE.Color` (linear) or they wash out. `CanvasTexture.colorSpace`
   must be `SRGBColorSpace`.
9. `mergeGeometries` returns `null` when parts mix indexed and non-indexed geometry or have different
   attributes. Normalize every part first.
10. Z-fighting: use the §6.7 heights; overlays use polygonOffset and `depthWrite: false`.
11. Leaks: shared geometries are disposed only in `disposeAll`. `InstancedMesh` needs
    `frustumCulled = false` or a bounding-sphere recompute.
12. Picking uses CSS px, not drawing-buffer px. Call `updateMatrixWorld` before raycasting. Tall
    objects hide what is behind them, so E2E uses `findClickPoint`.
13. Touch: Pointer Events only, a tap slop, `touch-action: none`, `overscroll-behavior: none`, no
    hover-only features.
14. Time: never `Date.now()` in core; rebase on clock rollback; remaining time is never negative.
    Status is always derived from timestamps; never store a "stage".
15. Randomness that affects game state (pupuk rolls) comes only from `state.rng` in core. Cosmetic
    randomness (wander, scenery) uses its own seeded PRNG in render.
16. The level-up bonus changes timestamps of many objects in one dispatch; `sceneSync` must handle many
    `objectChanged` events at once.
17. Withering also happens while the game is closed. That is intended; do not "fix" it.
18. Drag-to-act must never dispatch twice for the same target in one stroke.
19. Saves: validate everything, build from a whitelist, follow the §9.4 protocol, handle quota errors
    and unavailable storage.
20. The debug API must never ship: guard it with `__GB_DEBUG__` and check with `verify-bundle`.
21. CSP: no inline scripts or styles and no `setAttribute('style')` (`el.style.x = …` is fine).
22. Too many draw calls: one base mesh per object, instanced shadows and bubbles, merged scenery.
23. No emoji and no web fonts; locale and timezone are fixed in Playwright; `doctor` checks ICU.
24. Markdown is excluded from Prettier. Never paste logs into `todo.md`; reports go in their own files.
25. Work only on the current sprint. No drive-by refactors. Never "fix" a test by weakening it.

## Sprint 1: Toolchain, sandbox-safe scaffold, headless WebGL smoke

Goal: a project that installs, builds, lints, unit-tests and runs a WebGL smoke test in headless
Chromium inside the sandbox.

- [ ] Make `docs/` contain exactly two files: replace the template `docs/spec.md` with a short pointer
  ("The specification is the `## Specification` section of `todo.md`. Decisions are logged in
  `docs/decisions.md`.") and create `docs/decisions.md` (title, one line saying it is append-only, a
  first dated entry).
- [ ] Create `.npmrc`, `.prettierrc.json`, `.prettierignore` and `.editorconfig` exactly as in §2.3;
  append the §2.3 lines to `.gitignore` without removing existing lines; create `scripts/run.mjs` as
  in §2.2.
- [ ] Create `package.json` per §2.4 and §2.5 (engines, exact versions, every script including
  `pree2e`). Run `npm install` from the project root, keep `package-lock.json`, and confirm that
  `.npm-cache/` was used and `node_modules/` is ignored by git.
- [ ] Create `tsconfig.base.json`, `tsconfig.json`, `tsconfig.core.json`, `src/env.d.ts` and
  `src/core/env.d.ts` per §3.3.
- [ ] Create `vite.config.ts` per §2.6 and `vitest.config.ts` (node environment, `src/**/*.test.ts`,
  `allowOnly: false`, `define: { __GB_DEBUG__: 'true', __GB_VERSION__: '"test"' }`, v8 coverage
  into `coverage/`).
- [ ] Create `eslint.config.js` per §3.3 (ignores, layer bans, core purity, security selectors,
  unused-directive errors).
- [ ] Create `e2e/gl-profiles.json` and `playwright.config.ts` per §10.4.
- [ ] App shell: `index.html` (`lang="id"`, title "Gaskan Bertani", viewport meta with
  `viewport-fit=cover`, `<canvas id="scene">`, `<div id="ui">`, `<div id="fatal" hidden>`),
  `src/ui/styles.css` (full-viewport canvas, `overscroll-behavior: none`), `src/main.ts`, and
  `src/render/renderer.ts` (§6.1 settings, WebGL2 check with the §6.10 message, `ResizeObserver`,
  `requestRender()`). Temporary scene: orthographic camera, green ground plane, one orange box.
- [ ] Create `src/app/options.ts` (§10.2 flags, ignored unless `__GB_DEBUG__`), `src/debug/types.ts`
  (`Window.__GB__` typing) and `src/debug/debugApi.ts` with `ready`, `frameCount`, `renderNow`,
  `probePixel`, `getRenderInfo`; load it only through the dynamic import of §3.2. Add a placeholder
  `gallery.html` and `src/gallery/main.ts`.
- [ ] Create `scripts/doctor.mjs`, `scripts/verify-bundle.mjs` and `scripts/verify-hygiene.mjs` per
  §10.8.
- [ ] Create `e2e/support/test.ts`, `e2e/support/gb.ts` and `e2e/support/snap.ts` per §10.5 and
  §10.7 (helpers for later sprints may throw "not implemented yet" until their sprint).
- [ ] Tests: `src/core/smoke.test.ts`; `e2e/s01-webgl.spec.ts` (a WebGL2 context exists; log the
  renderer string; `__GB__.ready`; `probePixel` at the canvas centre is green-dominant;
  `snap('s01-smoke')`); `e2e/s01-fallback.spec.ts` (`?e2e=1&forceNoWebgl=1` shows `#fatal`
  containing "WebGL2").
- [ ] Run `npm run e2e:install` and `npm run doctor`. Make the first working GL profile the default
  (A, then B, then profile A with `GB_PW_CHANNEL=chromium`), and record the profile, the renderer
  string and the Chromium version in `docs/decisions.md`. If Chromium reports missing OS libraries or
  cannot launch inside the sandbox, write BLOCKED.md quoting §2.1.

**Acceptance**

- Inside the sandbox, `npm install`, `npm run e2e:install`, `npm run doctor`, `npm run check` and
  `npm run e2e` succeed, and nothing is written outside the project.
- `test-artifacts/screens/s01-smoke.png` shows green ground and an orange box on a light-blue
  background.
- `verify-bundle` passes on `dist/` (no debug API, CSP present).
- Out of scope: game rules, models, UI.

## Sprint 2: Core domain model, catalog, time and grid

Goal: the pure data model of the game, fully typed and tested.

- [ ] `src/core/types.ts` with every §3.5 type, names verbatim.
- [ ] `src/core/result.ts` (ok/err helpers), `clock.ts` (`Clock`, `ManualClock` with
  `now/advance/set`), `constants.ts` (§5.1 constants and `OBJECT_ID_RE`), `clone.ts` (`deepClone`,
  `deepFreeze`) and `prng.ts` (`mulberry32Step(state) → { value, state }` with `value` in [0, 1),
  `mulberry32(seed)` returning a generator for cosmetic use, `hashString`).
- [ ] `src/core/data/catalogTypes.ts` and `catalog.ts` with all §5 values as a deep-frozen `CATALOG`
  (economy, levels with rewards, crops, trees, animals, goods, buildings, starting plots).
- [ ] `src/core/data/validateCatalog.ts` per §5.9, with tests: `CATALOG` passes, and every rule has a
  test in which a modified copy fails.
- [ ] `src/core/levels.ts` (`levelForXp`, `xpProgress`, `levelReward`, `unlocksAtLevel`,
  `isUnlocked`, `plotLimit`) with tests for every threshold, the level-20 cap and XP beyond it.
- [ ] `src/core/water.ts` (`currentWater`, `nextWaterAt`, `spendWater`, `addWater`) per §4.6, with
  unit tests for the cap edges and a property test (≥ 300 seeded random sequences of spends, adds and
  time jumps) proving path independence, that regeneration never goes above `waterRegenCap`, and that
  the amount never exceeds `waterHardCap`.
- [ ] `src/core/grid.ts` (`footprintOf`, `inBounds`, `buildOccupancy`,
  `canPlace(state, catalog, footprint, x, y, ignoreId?)`) with tests for edges, overlaps, 2×2
  footprints, buildings and `ignoreId`.
- [ ] `src/core/state.ts` (`newGame(now, catalog, seed = 1)` per §5.8, `allocateId`) and
  `src/core/invariants.ts` (`checkInvariants` per §4.11), with tests: `newGame` passes; the starting
  farm matches §5.8 exactly; one test per invariant that violates it.
- [ ] `src/core/time.ts` (`shiftTimestamps`, `rebaseIfClockWentBack`) with a reflection test that every
  numeric `*At` and `updatedAt` field of a state containing every object kind is shifted, and a test
  that rebasing keeps every remaining duration.
- [ ] `src/core/testing/builders.ts`: helpers that build any object in any phase relative to `now`
  (for example `plotWith('tomat', 'ripe', now)`, `treeIn('mangga', 'ready', now)`,
  `animalIn('sapi', 'adultHungry', now)`), whose output passes `checkInvariants`; tests for each.

**Acceptance**

- `npm run check` is green; `tsc -p tsconfig.core.json` passes; at least 50 unit tests; the Sprint 1
  E2E is still green.
- Out of scope: command handlers, persistence, rendering.

## Sprint 3: Core rules I - plots, crops, goods, selling, placement, level-ups

Goal: the whole crop loop works in pure code, including withering, fertilizer and the level-up bonus.

- [ ] `src/core/rules/goods.ts`: `addGoods` (all or nothing against `GOOD_MAX`), `removeGoods`, no zero
  entries, events.
- [ ] `src/core/rules/plots.ts`: `plow`, `plant`, `waterPlot`, `harvestPlot`, `clearPlot`, `revive`,
  `fertilizePlot` per §4.3 and §4.10, each pushing the §3.5 events.
- [ ] `src/core/rules/market.ts` (`sell`) and `src/core/rules/placement.ts` (`move` for every kind
  including buildings; `remove` with the §4.9 refunds and `NOT_REMOVABLE`).
- [ ] `src/core/rules/levelUp.ts`: `applyLevelUps(beforeXp, draft, now, catalog, events)` with the
  rewards for every crossed level in order, then the one-time bonus of §4.2 with `levelBonus` and
  per-object `objectChanged` events.
- [ ] `src/core/commands.ts`: the `applyCommand` pipeline of §3.5; commands of Sprint 4 return
  `UNSUPPORTED` for now; exhaustive switch with a `never` check.
- [ ] `src/core/selectors.ts` part 1: `cropPhase`, `plotView` (ObjectView with stage, bubble,
  remainingMs, progress), `primaryAction` for plots, `hudModel` (koin, level, XP progress, water
  current/cap/nextAt, pakan, pupuk, botol), and the paint predicates `canWaterPlot`, `canHarvestPlot`,
  `canPlant`, `canPlow`, `canFertilize`.
- [ ] Tests `src/core/rules/*.test.ts`: every error code of these commands (the §4.10 rows) and their
  success effects and events, built with the builders.
- [ ] Tests `commands.test.ts`: deep-frozen state and catalog are never mutated; a rejected command
  returns the original state; crossing two levels gives two `levelUp` events with their rewards and one
  `levelBonus`; the bonus ripens watered growing and withered crops and watered trees but not seeded
  crops; water rewards may exceed `waterRegenCap` but never `waterHardCap`.
- [ ] Tests `selectors.test.ts`: stage boundaries at 0, g/3, 2g/3, g and g + w, each at −1 ms and
  +0 ms; `remainingMs` is never negative.
- [ ] Scenario `scenario.tutorial.test.ts`: `newGame` → harvest the 4 ripe tomatoes → water the 4
  seeded ones → plant tomat on the empty plot and water it → advance the clock by `growMs` → harvest all
  → sell → exact koin, XP and goods from the catalog; continue until level 2 and check its reward;
  invariants hold after every step.

**Acceptance**

- The §4.10 rows for plow, plant, water and harvest on plots, clearPlot, revive, fertilize on plots,
  sell, move and remove are implemented and tested.
- `npm run test:coverage` shows at least 90% line coverage for `src/core/rules`.
- Out of scope: trees, animals, bottles, well, mill, aid.

## Sprint 4: Core rules II - trees, animals, bottles, well, mill, aid

Goal: every command of §4.10 works in pure code.

- [ ] `src/core/rules/trees.ts` (`buyTree`, `waterTree`, `harvestTree`, `fertilizeTree`); route
  `water`, `harvest` and `fertilize` by target kind.
- [ ] `src/core/rules/animals.ts` (`buyAnimal`; `feed` for babies and adults per §4.5; the pupuk roll
  advances `state.rng` with `mulberry32Step`).
- [ ] `src/core/rules/supplies.ts` (`buyBottles`) and `src/core/rules/well.ts` (`collectWell`).
- [ ] `src/core/rules/mill.ts` (`mill`) and `src/core/rules/aid.ts` (`isAidAvailable`, `claimAid`)
  per §4.8.
- [ ] Finish `applyCommand`: every `Command` variant is handled; `UNSUPPORTED` remains only for
  unknown types. Enable the §10.1 coverage thresholds in `vitest.config.ts`.
- [ ] Selectors part 2: `treeView`, `animalView`, `buildingView`, `objectView` (by kind),
  `primaryAction` for every kind (§8.2), `objectInfo` (display name including baby names, §8.6 status
  key and params, remainingMs, progress, allowed actions), `aidAvailable`.
- [ ] Tests `trees.test.ts`: first-cycle stages 0 → 1 → 2 → 3; harvesting requires watering again;
  fruit is `unripe` at p ≥ 0.5 on mature trees; a 30-day absence leaves exactly one ready harvest and
  nothing withers; a fertilized harvest gives +1; every error code.
- [ ] Tests `animals.test.ts`: the bottle schedule and growing up at `bottlesToAdult`; a new adult is
  hungry immediately; feeding gives the product at once and starts the cooldown; every error code;
  `maxOwned`; with fixed seeds the pupuk roll is deterministic, respects `fertilizerCap`, and over
  10,000 seeded feeds its rate is within ±2% of each `fertilizerChance`.
- [ ] Tests `well.test.ts`, `mill.test.ts` (`FEED_FULL` with `detail.max`), `supplies.test.ts` and
  `aid.test.ts` (each condition and the cooldown).
- [ ] Property test `rules.property.test.ts`: at least 500 seeded random sequences of commands and time
  jumps (backwards jumps included): `applyCommand` never throws, invariants always hold, a rejected
  command never changes the state, and a plot's phase only moves forward in time (seeded → growing →
  ripe → withered) unless a command changes it.
- [ ] Scenario `scenario.animals.test.ts`: from a level-3 builder state with enough koin: buy an ayam →
  buy 2 bottles → bottle it, advance `bottleIntervalMs`, bottle again → adult → mill tomatoes into
  pakan → feed → telur → sell; exact numbers from the catalog.

**Acceptance**

- Every `Command` is implemented and covered; the property tests pass; `npm run check` is green and
  `npm run test:coverage` meets the thresholds.

## Sprint 5: Persistence, store, fixtures and the debug state API

Goal: the game state survives reloads safely and E2E tests can control it.

- [ ] `src/core/save/schema.ts`: `validateSave` per §9.3.
- [ ] `src/core/save/migrations.ts` and `serialize.ts` per §9.2 and §9.4, plus the golden file
  `src/core/save/__fixtures__/v0-basic.json`.
- [ ] `src/app/persistence.ts` per §9.1 and §9.5 (`StorageLike`, backup, quarantine, quota handling,
  fallback chain, `LoadOutcome`).
- [ ] `src/app/systemClock.ts` and `src/app/store.ts` per §3.6 (effective time with rebase, 400 ms
  autosave debounce, `flushSave`, `tick`).
- [ ] `src/debug/fixtures.ts` with every §10.6 fixture.
- [ ] `src/app/game.ts`, the composition root used by `main.ts`: load on boot, frozen `ManualClock`
  under e2e, `?fixture=`, flush on `visibilitychange → hidden` and `pagehide`.
- [ ] Extend the debug API: `getState`, `getCatalog`, `dispatch`, `fastForward`, `setNow`, `reset`,
  `flushSave`, `getLoadOutcome`, `getEvents`.
- [ ] Unit tests: `schema.fuzz.test.ts` (≥ 1000 seeded mutations of valid saves; validation never
  throws; every accepted state passes `checkInvariants`); pollution payloads (`__proto__`,
  `constructor`, `prototype` as keys and ids) are rejected and `Object.prototype` stays clean; NaN,
  Infinity, negative, fractional and huge numbers are rejected; `migrations.test.ts` (the v0 golden file
  migrates, future versions are refused); `persistence.test.ts` (quota, unavailable storage, backup,
  quarantine cap); `store.test.ts` (debounce, rollback rebase, rejected dispatch);
  `fixtures.test.ts`.
- [ ] E2E `e2e/s05-persistence.spec.ts`: plow + plant → `flushSave` → reload gives a deep-equal
  state; `fastForward` past `growMs` → reload keeps the crop ripe; `fastForward` past
  `growMs + witherMs` → the crop is withered after reload; a corrupt main save with a valid backup gives
  `recoveredFromBackup`; both corrupt give `resetCorrupt` with the quarantine present and the game
  still playable.

**Acceptance**

- Reloads preserve the state exactly; no localStorage content can crash the boot; `npm run check:all`
  is green.
- Out of scope: tab guard, toasts and banners (Sprint 16).

## Sprint 6: Render foundation - camera, coordinates, input, tile picking

Goal: an isometric farm ground you can pan, zoom and click with exact tile picking.

- [ ] `src/render/coords.ts` (§6.3) and `src/render/camera.ts` with `IsoCameraRig` (§6.2).
- [ ] `src/render/input/gestures.ts` (pure recognizer with tap, pan, pinch, wheel and paint events and
  the `canPaintAt` callback) and `src/render/input/pointerInput.ts` (DOM binding) per §6.4.
- [ ] `src/render/picking.ts` (tile picking now; the API is ready for objects and bubbles).
- [ ] `src/render/ground.ts` (farm tiles with seeded colour variation, outer grass), `layers/grid.ts`,
  `layers/highlight.ts`, `lighting.ts` (§6.8) and `scene.ts`; remove the temporary scene.
- [ ] Render loop basics (§6.10): render on demand, continuously only during gestures, stopped while
  hidden.
- [ ] Extend the debug API: `tileToScreen`, `screenToTile`, `pickAt`, `getCamera`, `setCamera`,
  `focusTile`.
- [ ] Unit tests: `coords.test.ts` (round-trip all 576 tiles at zoom 0.4, 1 and 2.5 with aspect 16:9
  and 9:19.5); `camera.test.ts` (`zoomAt` keeps the point under the cursor fixed within 1e-6; clamps);
  `gestures.test.ts` (tap; slop → pan; paint when `canPaintAt` is true; a second pointer cancels paint
  and starts a pinch; pinch then lift gives no tap; right button pans; `pointercancel`).
- [ ] E2E `e2e/s06-camera.spec.ts`: 12 seeded random tile clicks where `pickAt` and the highlight both
  match; a drag pans and clamps at the edges; wheel zoom clamps; `frameCount` is unchanged across 60
  animation frames while idle; snaps `s06-ground`, `s06-zoom-in`, `s06-zoom-out`.

**Acceptance**

- 100% tile-picking accuracy; the camera never leaves its bounds; nothing renders while idle.

## Sprint 7: Model kit, plot and crop models, gallery v1

Goal: the procedural model system and every plot and crop model.

- [ ] `src/render/models/palette.ts` (§7.2) and `materials.ts` (§6.6).
- [ ] `src/render/models/kit.ts` (`ModelBuilder`, `ModelData`) per §6.6.
- [ ] `src/render/models/library.ts` (`ModelLibrary`: `get`, `has`, `keys`, `stats`, `disposeAll`).
- [ ] `src/render/models/plot.ts` (`plot:empty`, `plot:seeded`, `plot:withered`, the wet-bed helper)
  and `src/render/models/overlays.ts` (`fx:fertilized`).
- [ ] Crop models `src/render/models/crops/tomat.ts`, `padi.ts`, `cabai.ts`, `jagung.ts`,
  `singkong.ts` and `bawangMerah.ts` per §7.3, with keys `plot:<crop>:1` to `plot:<crop>:4`.
- [ ] `src/render/view.ts`: `modelKeysFor(visual)` for plots (base key plus optional overlay key).
- [ ] Gallery v1 (`src/gallery/main.ts`): a category grid with DOM labels; flags `cat`, `key`, `zoom`;
  `window.__GALLERY__ = { ready, keys }`.
- [ ] Unit tests `models.test.ts`, for every key: it builds; it is non-indexed with exactly the
  position, normal and color attributes; no NaN; the bounding box is inside the footprint ±0.05;
  triangles within §6.11; crop heights strictly increase from stage 1 to 4; stage 4 contains its ripe
  palette colour on at least 10 triangles; `plot:withered` contains `wither`. Also: the cache returns
  the same instance and `disposeAll` disposes everything.
- [ ] E2E `e2e/s07-gallery.spec.ts`: no console errors; snaps `s07-plots`, `s07-crops` and
  `s07-<crop>-4` for each of the 6 crops.

**Acceptance**

- §7.6 items 1, 2, 3 and 8 pass on the screenshots; the budgets pass.

## Sprint 8: Tree and animal models, gallery v2

Goal: every tree stage with unripe and ripe fruit, and every animal as baby and adult.

- [ ] `src/render/models/trees/treeKit.ts` (trunk, canopy clusters, deterministic fruit placement on the
  canopy surface, sapling helper).
- [ ] `jeruk.ts`, `mangga.ts`, `rambutan.ts` and `durian.ts` (stages 0–3, `fruit-unripe`,
  `fruit-ripe`) per §7.3.
- [ ] `kelapa.ts` (the palm) per §7.3.
- [ ] `src/render/models/animals/ayam.ts`, `kambing.ts` and `sapi.ts` (baby and adult, with `anchor`
  and `headZ` metadata).
- [ ] Register the keys `tree:<id>:0` to `tree:<id>:3`, `tree:<id>:fruit-unripe`,
  `tree:<id>:fruit-ripe`, `animal:<id>:baby` and `animal:<id>:adult`; extend `view.ts` (fertilized
  trees use `fx:fertilized` scaled ×2).
- [ ] Gallery v2: categories `trees` and `animals`.
- [ ] Tests: budgets; tree bounding boxes within 2×2 ±0.1 with height increasing over stages 0–3; every
  `fruit-ripe` key has its fruit colour on at least 12 triangles and every `fruit-unripe` key has
  `unripeGreen` (or `coconut` for kelapa); `headZ` is greater than the body centroid's z; a baby's
  bounding-box volume is less than half its adult's.
- [ ] E2E `e2e/s08-gallery.spec.ts`: snaps `s08-trees`, `s08-animals` and `s08-<tree>-fruit` for all 5
  trees.

**Acceptance**

- §7.6 items 4, 5 and 8 pass; the budgets pass.

## Sprint 9: Buildings, scenery, icons, bubbles and shadows, gallery v3

Goal: the remaining art: buildings, the world around the farm, icons and status bubbles.

- [ ] `src/render/models/buildings/rumah.ts`, `gudang.ts`, `penggilingan.ts` and `sumur.ts`
  (`building:sumur:full` and `building:sumur:empty`) per §7.3.
- [ ] `src/render/models/decorations.ts` with every §7.3 decoration.
- [ ] `src/render/scenery.ts` (§7.4: seeded, merged, one draw call).
- [ ] `src/icons/atlas.ts` and `src/icons/draw.ts` with all 26 icons (§7.5).
- [ ] `src/render/iconTexture.ts` (atlas `CanvasTexture`, sRGB, per-icon clones sharing one source).
- [ ] `src/render/layers/bubbles.ts` (§6.7) and `layers/blobShadows.ts`.
- [ ] Gallery v3: categories `buildings`, `decor`, `scenery`, `icons` and `layers` (demo bubbles and
  shadows at two zoom levels); `__GALLERY__.bubbleScreenSize()`.
- [ ] Tests: `atlas.test.ts` (no overlaps, UVs in [0, 1], every IconId present); building and
  decoration budgets and bounding boxes; `scenery.test.ts` (deterministic per seed; no vertex inside the
  farm except the fence band); `bubbles.test.ts` (capacity growth, instance counts).
- [ ] E2E `e2e/s09-gallery.spec.ts`: snaps `s09-buildings`, `s09-decor`, `s09-scenery`, `s09-icons`
  and `s09-layers`; the bubble screen size is equal (±2 px) at two zoom levels.

**Acceptance**

- §7.6 items 6, 7, 8 and 9 pass; the scenery is one draw call.

## Sprint 10: UI foundation - HUD, toolbar, toasts, floating text, panels

Goal: the DOM overlay with the HUD, feedback and the Seed Picker, all in Bahasa Indonesia.

- [ ] `src/ui/dom.ts` (`el()` using `textContent` only, listener disposers), `strings.id.ts` (§8.6
  and §8.7) and `format.ts` (durations and numbers).
- [ ] `src/ui/ports.ts` (§3.6) and `viewModels.ts` (HUD, seed list with lock and affordability).
- [ ] `src/ui/styles.css`: design tokens, buttons, pills, panel and bottom sheet (§8.1).
- [ ] `src/ui/hud.ts` with the §8.8 test ids; it updates on every store change and on a 1 Hz timer
  outside e2e.
- [ ] `src/ui/toolbar.ts` and `src/ui/modeChip.ts`; tools that arrive in later sprints show the toast
  "Segera hadir".
- [ ] `src/ui/toast.ts` (rejected dispatches mapped through §8.7 with detail placeholders) and
  `floatingText.ts` (§8.5).
- [ ] `src/ui/panel.ts` (one panel at a time; Esc, backdrop and Tutup close it) and
  `panels/seedPicker.ts` (§8.4).
- [ ] Wire everything in `app/game.ts` with a temporary projector (footprint centre); extend the debug
  API with `ui.open`, `ui.close` and `getUiState`.
- [ ] Unit tests: `format.test.ts` (boundaries at 59 s, 60 s, 59 m, 60 m and 24 h; id-ID numbers);
  `strings.test.ts` (every ErrorCode and every status key has text, placeholders are filled);
  `viewModels.test.ts`.
- [ ] E2E `e2e/s10-ui.spec.ts`: the HUD shows the `midgame` values; a debug harvest updates the HUD
  and shows floating text; watering in `noWater` shows the §8.7 toast; Seed Picker → `seed-tomat`
  plants and deducts koin; a locked crop is disabled and shows its level; snaps `s10-hud` and
  `s10-seed-picker`.

**Acceptance**

- All UI text is Indonesian; no `innerHTML`; the HUD is exact after every tested action.

## Sprint 11: Scene sync, object picking, smart tap - first playable loop

Goal: the farm is drawn from the state, and clicking objects plays the game.

- [ ] `src/render/sceneSync.ts`: reconcile objects by id and model keys with shared geometry; swap
  geometry when a key changes; deterministic tree yaw; overlays (fruit, fertilized); shadows and
  bubbles from `objectView`; `getPickables`; `projectObject` (model anchor).
- [ ] Complete `view.ts`: `modelKeysFor` for every kind and `bubbleIconFor`.
- [ ] Picker per §6.5: bubbles → meshes and animal proxies → tile.
- [ ] `src/app/uiState.ts` and `src/app/interaction.ts` (normal mode, every row of §8.2).
- [ ] `src/ui/panels/objectInfo.ts` (Pindahkan and Jual stay disabled until Sprint 12) and
  `src/ui/modals/withered.ts` (Bersihkan → `clearPlot`; Pulihkan → `revive`, showing the cost).
- [ ] `src/ui/tooltip.ts`: the desktop hover tooltip (mouse only) with name, status and time left,
  refreshed at 1 Hz outside e2e.
- [ ] Wiring: store changes and the 1 Hz tick → `sceneSync.sync` → `requestRender`; events → floating
  text at `projectObject`. Gudang and Penggilingan taps open stub panels until Sprint 13; tapping a full
  sumur already collects water.
- [ ] Extend the debug API: `objectToScreen`, `findClickPoint`, `getBubbles`, `profileSync`.
- [ ] Unit tests: `interaction.test.ts` (every §8.2 row with fake ports); `view.test.ts` (every
  reachable VisualState and bubble maps to model keys and an icon).
- [ ] E2E `e2e/s11-play.spec.ts`: on `fresh`, click a ripe tomato → harvested; click a seeded plot →
  watered; click the empty plot → Seed Picker → Tomat; `fastForward` → ready bubble → click harvests;
  goods and floating text are correct. On `midgame`: water a tree by clicking its canopy; feed a baby
  (bottle) and an adult (pakan); collect the well; a withered plot → modal → Pulihkan; flush, reload,
  and the ids and model keys are identical. Snaps `s11-fresh`, `s11-allstages` and `s11-tooltip`.

**Acceptance**

- The crop loop is playable by clicking alone (plus `fastForward`); on `allStages`, draw calls are
  ≤ 2 × objects + 20.

## Sprint 12: Modes - shop, placement, seed, plow, fertilize, move, sell, drag-to-act

Goal: every tool of the toolbar and FarmVille 2's drag-to-act work.

- [ ] `src/ui/panels/shop.ts` (§8.4: tabs Benih, Pohon, Hewan, Perlengkapan with Botol Susu ×1, ×5,
  ×10).
- [ ] `src/render/layers/ghost.ts` (§6.6, §6.7) and the grid overlay in place, move and plow modes.
- [ ] Modes `seed`, `plow`, `fertilize`, `move`, `sell` and `place` in `interaction.ts` per §8.3;
  `src/ui/placementBar.ts`; the toolbar buttons Bajak, Pupuk, Pindah and Jual become active; the mode
  chip shows the mode and "Batal".
- [ ] Drag-to-act per §8.3: `canPaintAt` for each mode; per-stroke dedupe by object id or tile; one
  resource toast ends the stroke.
- [ ] Object info actions: Pindahkan → move mode; Jual or Hapus → `modals/confirm.ts` → `remove`;
  Beri Pupuk when allowed.
- [ ] `getUiState()` reports `mode`, `seedCropId`, `ghost { x, y, valid, reason }` and `painting`.
- [ ] Unit tests `interaction.modes.test.ts`: entering and leaving every mode; invalid tiles never
  dispatch; plow mode stays active after `LIMIT_REACHED`; seed mode exits when koin run out (toast);
  move ignores the object itself; the sell confirmation shows the refund; Esc.
- [ ] Unit tests `interaction.paint.test.ts`: a stroke over N targets dispatches exactly once per valid
  target; harvest strokes skip unripe plots; water strokes stop at the first `NOT_ENOUGH_WATER` with
  one toast; a stroke that starts on an invalid target pans instead.
- [ ] E2E `e2e/s12-modes.spec.ts`: plow 3 plots; an occupied tile shows an invalid ghost and a toast
  without a dispatch; on `midgame`: place a jeruk tree and an ayam from the Toko; buy 5 bottles; move a
  plot and a building; sell a tree (confirm, exact refund); fertilize a plot; snaps `s12-ghost-valid`,
  `s12-ghost-invalid` and `s12-shop`.
- [ ] E2E `e2e/s12-paint.spec.ts`: drag across a 3×3 field: seed mode plants all 9; a normal-mode
  stroke from a seeded plot waters all; after `fastForward` a stroke harvests all; a stroke with too
  little air stops with exactly one toast.

**Acceptance**

- Every kind of object can be bought, placed, moved and sold through the UI; drag-to-act works for
  plant, water, harvest, plow and fertilize.

## Sprint 13: Building panels, level-up and aid - complete loop

Goal: the full economy is playable through the UI.

- [ ] `src/ui/panels/storage.ts` (Gudang: goods list, quantity 1 / 5 / Semua, Jual with the total,
  the resource summary).
- [ ] `src/ui/panels/mill.ts` (Penggilingan: ratios, quantity 1 / 5 / Maks, Giling, pakan bar) and
  `panels/well.ts` (Sumur: status, Ambil Air, regeneration line).
- [ ] `src/ui/modals/levelUp.ts`: queue, rewards, unlocked items, and the bonus line when `levelBonus`
  ripened or revived anything.
- [ ] Wire the building taps and the toolbar's Gudang button to the real panels; the aid banner with
  its claim button (§8.5).
- [ ] View-models for the Gudang, mill and well panels (maximum quantities, totals, reasons a control
  is disabled), with unit tests.
- [ ] E2E `e2e/s13-loop.spec.ts`: from `fresh`, play the FarmVille 2 tutorial loop through the UI:
  harvest 4, water 4, plant 1, `fastForward`, harvest, sell in the Gudang (exact koin), reach level 2
  → the level-up modal shows the rewards and unlocks. On `levelUp`: harvesting one tomato triggers the
  bonus, the watered crops become ripe and the withered one is revived.
- [ ] E2E `e2e/s13-economy.spec.ts`: mill tomatoes into pakan (exact amount; quantities beyond the max
  are disabled); collect the well (+10 air) and see the refill time; on `softlock` the aid banner gives
  50 koin once and then hides; on `noBottles` feeding a baby shows the bottle toast, and after buying
  bottles in Perlengkapan feeding works.
- [ ] Every loop test ends with a reload and checks that the state is preserved; snap every panel and
  the level-up modal.

**Acceptance**

- The whole §1 loop (crops, trees, animals, well, mill, selling, levels) is playable through the UI.

## Sprint 14: Animation, juice, sound and render lifecycle

Goal: the farm feels alive, with no cost while idle and nothing animated under e2e.

- [ ] `src/render/anim/tweens.ts` (no dependencies) and `src/render/anim/effects.ts` (pop, harvest
  squash, bubble bob, fertilized sparkle twinkle, pan inertia per §6.9).
- [ ] `src/render/anim/wander.ts` (§6.9) integrated into `sceneSync`; shadows and bubbles follow the
  animal.
- [ ] Harvest fly to the Gudang: `floatingText.ts` gains `flyToStorage(goodId, from)` (an atlas icon on
  a small canvas, CSS transform, 600 ms).
- [ ] `src/render/anim/scheduler.ts`: continuous rendering only while something animates; pause and
  resume per §6.10.
- [ ] WebGL context loss and restore with the overlay from `src/ui/overlay.ts` (§6.10).
- [ ] `src/audio/sfx.ts`: WebAudio-synthesized sounds (tap, plow, plant, water, harvest, coins,
  levelUp, error, well, bottle, fertilize, cluck, mbek, moo); the AudioContext is created on the first
  user gesture; a no-op under e2e; a pure `soundForEvent(event)` mapping used by `app/game.ts`.
- [ ] `src/app/settings.ts` (§9.1 settings key, whitelist-validated) and `src/ui/panels/settings.ts`
  with the Suara and Kurangi Animasi toggles (Mulai Ulang arrives in Sprint 16).
- [ ] The `?perf=1` overlay (debug builds only): fps, draw calls, triangles.
- [ ] Unit tests: `wander.test.ts` (10k steps × 5 seeds stay inside the footprint; deterministic),
  `tweens.test.ts`, `sfx.test.ts` (the `soundForEvent` mapping) and `settings.test.ts` (validation).
- [ ] E2E `e2e/s14-lifecycle.spec.ts`: with `anim=1` an animal moves but stays inside its footprint;
  in plain e2e `frameCount` is unchanged while idle; losing the context with `WEBGL_lose_context` and
  restoring it shows the ground again (`probePixel`); the settings toggles persist across a reload.

**Acceptance**

- No animation and no sound under e2e; rendering stops when idle; context loss recovers; reduced motion
  disables every animation.

## Sprint 15: Mobile and responsive UX

Goal: the complete game works with touch on a phone, in portrait and landscape.

- [ ] Responsive CSS per §8.1: bottom sheets, a compact HUD, safe areas, `100dvh`, 44 px touch
  targets, a landscape-phone layout, a horizontally scrollable toolbar when narrow.
- [ ] Touch tuning per §6.4: pinch anchor; two-finger pan; pinch then one finger gives neither a tap
  nor paint; one-finger paint strokes on plots; scrollable panels use `touch-action: pan-y`.
- [ ] Orientation changes and resizes: recompute the frustum, reflow panels, keep the ghost on its
  tile.
- [ ] The touch placement flow (a tap positions the ghost, "Pasang" confirms). Nothing relies on
  hover; the tooltip's information is available in object info.
- [ ] Unit tests: more touch cases in `gestures.test.ts` (`pointercancel` during a pinch and during a
  paint stroke).
- [ ] E2E `e2e/s15-touch.mobile.spec.ts`: tap to plant, water and harvest; a one-finger drag waters 3
  plots; CDP pinch in and out changes the zoom within its limits; a one-finger drag on grass pans;
  panels scroll and close; a Toko placement is confirmed with Pasang.
- [ ] E2E `e2e/s15-layout.mobile.spec.ts`: the HUD, mode chip and toolbar do not overlap in portrait
  and landscape (bounding boxes); snaps `s15-portrait`, `s15-landscape` and `s15-sheet`.

**Acceptance**

- The complete loop is playable in the mobile project with no overlapping UI.

## Sprint 16: Hardening, onboarding and release

Goal: the project Definition of Done (§11.3).

- [ ] E2E `e2e/s16-perf.spec.ts`: the §6.11 budgets on `stress`; a leak test of 50 ×
  (`reset('stress')`, `reset('fresh')`) keeps geometries and textures within the budget.
- [ ] `src/app/tabGuard.ts` (§9.6) with the two-page E2E `e2e/s16-tabguard.spec.ts`.
- [ ] Storage edge cases (`e2e/s16-storage.spec.ts`): an init script that makes localStorage throw →
  the in-memory banner "Progres tidak dapat disimpan"; a `setNow` rollback → the rebase keeps the
  remaining times; a future-version save → the read-only banner and the save is untouched.
- [ ] Onboarding `src/ui/hints.ts` (§8.5 steps, progress in the settings key; off under e2e unless
  `hints=1`) with `e2e/s16-onboarding.spec.ts` covering the whole first-run flow on `fresh`.
- [ ] Settings panel: add Mulai Ulang (confirmation → new game) and the version (`__GB_VERSION__`).
- [ ] Security pass per §10.8; raise the save fuzz to 5000 iterations; `npm audit --omit=dev` clean.
- [ ] `README.md`: what the game is, requirements, commands, URL flags, SSH-tunnel viewing, and a
  short "Cara bermain" section in Bahasa Indonesia; a final summary entry in `docs/decisions.md`.
- [ ] Full regression with `npm run check:all`; review every screenshot against §7.6 and fix the
  defects found.

**Acceptance**

- The §11.3 project Definition of Done is met.
