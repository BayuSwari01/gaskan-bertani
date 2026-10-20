# §10 Testing

## §10.1 Layers

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

## §10.2 URL flags (honoured only when `__GB_DEBUG__` is true)

| Flag | Effect |
|---|---|
| `e2e=1` | frozen `ManualClock` (starts at real now); no 1 Hz tick; no animations, CSS transitions, sounds, hints or tab guard; DPR 1; antialias off; cosmetic seed 1; static floating texts |
| `fixture=<name>` | boot with that fixture (§10.6) |
| `seed=<int>` | cosmetic random seed |
| `anim=1`, `hints=1`, `tabguard=1` | turn that feature back on under e2e |
| `forceNoWebgl=1` | show the WebGL fallback |
| `perf=1` | performance overlay |
| `cat`, `key`, `zoom` | gallery only |

## §10.3 Debug API (`window.__GB__`, debug builds only, typed in `src/debug/types.ts`)

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

## §10.4 Playwright

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

## §10.5 E2E conventions

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

## §10.6 Fixtures (`src/debug/fixtures.ts`; times relative to now; every fixture passes `validateSave`)

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

## §10.7 Screenshots

- `snap(page, 'sNN-name')` writes `test-artifacts/screens/sNN-name.png` after `renderNow()`. Names are
  stable, so later runs overwrite them.
- No pixel-golden assertions. Visual quality is reviewed by the tester against §7.6. Objective checks:
  model unit tests (palette colours, triangle counts) and `probePixel` colour probes.

## §10.8 Verification scripts and security checklist

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
