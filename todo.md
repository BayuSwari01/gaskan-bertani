# TODO — Gaskan Bertani

Gaskan Bertani is a single-player browser game in Three.js low-poly 3D. It clones the core loop of
FarmVille 2: crops on plots, fruit trees and animals. All in-game text is Bahasa Indonesia; code,
comments, tests and docs are English.

How to use this file:

- The specification lives in `docs/`; start with `docs/README.md`. Sprint tasks refer to it as §N,
  and §N is in `docs/spec/NN-*.md` (for example §4.3 is in `docs/spec/04-game-rules.md`).
- Before your first task, read `docs/game-design.md` (why the game works this way, with worked
  examples), `docs/visual-guide.md` (what the player sees) and `docs/glossary.md` (Indonesian terms).
- The coder works on exactly one `## Sprint N` per loop and checks off its tasks. The reviewer adds
  `### Fix` items under a sprint when something must be fixed.
- Balance numbers live only in `src/core/data/catalog.ts`, which mirrors §5. Tests read numbers from
  the catalog and never hard-code them.
- Code follows `docs/spec/`. When the specification is silent or ambiguous, choose the simplest option
  that is consistent with it and append a dated line to `docs/decisions.md`; when an implementation
  must deviate, update the affected spec lines in the same commit and log the reason there.
- Checkboxes appear only in the sprints below.

## Sprint 1: Toolchain, sandbox-safe scaffold, headless WebGL smoke

Goal: a project that installs, builds, lints, unit-tests and runs a WebGL smoke test in headless
Chromium inside the sandbox.

- [ ] Read `docs/README.md` and every file it lists. `docs/` is already complete: do not restructure it.
  If a template placeholder `docs/spec.md` exists (from the my-ai-agent template), delete it; the
  specification is `docs/spec/`. Append a dated line to `docs/decisions.md` saying the project
  scaffold started.
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
