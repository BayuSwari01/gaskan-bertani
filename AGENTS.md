# Gaskan Bertani

Gaskan Bertani is a single-player browser farming game that clones the core loop of FarmVille 2:
crops on plots (with withering and fertilizer), fruit trees, and animals (ayam, kambing, sapi) that are
raised with milk bottles and then fed to produce goods. It is built with Three.js in low-poly 3D under a
fixed isometric camera, runs offline, saves to localStorage, and all in-game text is Bahasa Indonesia.

The specification lives in `docs/` (index: `docs/README.md`). `docs/spec/NN-*.md` holds §1–§12, the
normative rules; `docs/game-design.md`, `docs/visual-guide.md`, `docs/glossary.md` and
`docs/research-farmville2.md` explain the intent; `docs/decisions.md` is the append-only decision log.
The work plan is `todo.md`: sprints of checkbox tasks that refer to the specification as §N.

## Read first, every session

1. This file.
2. `docs/README.md`, then every file in `docs/` (the game design and visual guide first, then
   `docs/spec/` §1–§12, then `docs/decisions.md`).
3. `todo.md`: the header and your assigned sprint, including its `### Fix` items.
4. The reports your role needs (`coder-report.md`, `tester-report.md`).

## Tech stack

- TypeScript 6.0.3 (strict; never 7.x), Vite 8.3.4, three 0.186.1 (the only runtime dependency).
- Vitest 5.0.5 for unit tests; Playwright 1.64.1 for E2E in headless Chromium with SwiftShader WebGL.
- ESLint 10 with typescript-eslint 8.72, Prettier 3.9. Exact versions in §2.4.
- Node `^22.13.0 || ^24.0.0 || >=26.0.0`. No other dependencies (§2.4, §11.1).

## Commands (§2.5)

- `npm run check`: typecheck, lint, format check, unit tests, production build, bundle and hygiene
  checks. No browser needed.
- `npm run e2e`: builds the e2e bundle and runs Playwright; it installs Chromium into `node_modules`
  first. One file: `npm run e2e -- e2e/s06-camera.spec.ts`.
- `npm run check:all`: both of the above. It must be green at the end of every sprint.
- `npm run test:coverage`: unit tests with the coverage thresholds (from Sprint 4).
- `npm run doctor`: checks Node, the caches, Chromium and the WebGL renderer.
- `npm run dev`: dev server on 127.0.0.1:5317, for humans. Stop every server you start.
- Environment variables: `GB_GL_PROFILE` (`A` or `B`), `GB_PW_CHANNEL` (for example `chromium`),
  `GB_PW_WORKERS`.

## Sandbox rules (§2.2)

- You may write only inside the project folder. Run npm only from the project root; it uses the
  project-local `.npm-cache/`.
- Run tools only through the npm scripts or `node scripts/run.mjs <bin>`, which keep temp files,
  HOME and the Playwright browsers inside the project. Never run `npx playwright` directly, never
  `npm i -g`, never `sudo`, never write to `~` or `/tmp`.
- Write `BLOCKED.md` only for problems the owner must fix: Chromium OS libraries missing, no network,
  a browser that cannot start inside the sandbox. Include the exact error and the command from §2.1,
  then stop.

## Rules

- Follow the specification. Do not add features, screens, dependencies or files the plan does not ask
  for. Work only on the assigned sprint; no drive-by refactors.
- Layers (§3.2): `src/core` is pure (no three, no DOM, no `Date.now`, no `Math.random`, no timers).
  `render` and `ui` never import `app`. Every game state change goes through `applyCommand`.
- Status, stages and timers are always derived from timestamps (§3.7). Randomness that changes the
  game state comes only from `state.rng`.
- Balance numbers live only in `src/core/data/catalog.ts` (§5). Tests read them from the catalog.
- All UI text lives in `src/ui/strings.id.ts` and is Bahasa Indonesia; display names live in the
  catalog. Code, comments, tests and docs are English.
- DOM: build elements with `textContent`. Never use `innerHTML`, `outerHTML`, `insertAdjacentHTML`,
  `eval`, `new Function` or `setAttribute('style')`.
- Saves are untrusted input: validate them with the whitelist validator (§9.3). After Sprint 5, every
  change to the shape of `GameState` follows §9.4 (new version, migration, golden file).
- The debug API (`window.__GB__`) and the gallery exist only in debug and e2e builds; `dist/` must
  never contain them (`npm run verify`).
- Files stay under 300 lines; no `any`, no `as unknown as`, no non-null `!`, no enums.
- When the specification is silent, choose the simplest consistent option and append a dated line to
  `docs/decisions.md`.
- `docs/spec/` wins over every other document. Code follows the spec; never rewrite the spec to match
  code. Change it only when a task or Fix item asks, or when a deviation is unavoidable; then update
  the affected lines in the same commit and log the reason in `docs/decisions.md` (append-only).
- Never put checkboxes in `docs/`; tasks belong in `todo.md`.

## Testing rules

- Every task adds or updates tests. Never delete, skip or weaken a test to make it pass, and never
  commit `.only` or `.skip`.
- E2E: import `test` and `expect` from `e2e/support/test.ts`; never use `waitForTimeout`; move time
  only with `gb.fastForward()` using durations from `gb.catalog()`; click with `gb.clickTile` and
  `gb.clickObject`; use the fixtures of §10.6.
- Screenshots go to `test-artifacts/screens/sNN-name.png` (gitignored) and are reviewed against the
  §7.6 checklist.

## Role notes

- Coder: implement the assigned sprint's unchecked tasks and Fix items, run `npm run check:all`
  (and `npm run test:coverage` from Sprint 4), and list in `coder-report.md` the commands you ran with
  their result, the tests you added, the screenshots produced and any decisions.
- Tester: run `npm run doctor`, `npm run check:all` and `npm run test:coverage`; look at every new
  screenshot against §7.6; check that earlier sprints still work; run the §10.8 security checklist.
  The last line of `tester-report.md` is exactly `OK` or exactly `NOT OK`.
- Reviewer: check the sprint's Acceptance list and the §11.2 sprint Definition of Done, and check
  that any change to `docs/spec/` is justified in `docs/decisions.md`. Write `DONE.md` only when the
  §11.3 project Definition of Done holds.
- Git: the loop commits after each role. Never rewrite history. Generated folders (`node_modules`,
  `.npm-cache`, `.tmp`, `dist`, `dist-e2e`, `test-artifacts`, `coverage`, `.vitest`) stay gitignored.
