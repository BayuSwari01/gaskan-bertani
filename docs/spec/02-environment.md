# §2 Environment, tooling and commands

## §2.1 Owner prerequisites (one-time, done by the human; agents cannot do these)

- Linux x86-64 or arm64: Debian 12/13 or Ubuntu 22.04/24.04/26.04. No GPU and no display needed.
- Node `^22.13.0 || ^24.0.0 || >=26.0.0` (24 LTS recommended) with npm 10 or newer, and git.
- Chromium OS libraries, once, as root: `sudo npx -y playwright@1.64.1 install-deps chromium`.
- Outbound internet access for npm packages and the Chromium download.
- To view the game from another machine, use an SSH tunnel, for example
  `ssh -L 5317:127.0.0.1:5317 <server>`, then open http://127.0.0.1:5317. Servers never bind to
  `0.0.0.0`.

## §2.2 Sandbox rules (agents may write only inside the project folder)

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

## §2.3 Config files

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

## §2.4 Dependencies (exact versions, nothing else)

- Runtime: `three@0.186.1`.
- Dev: `@eslint/js@10.0.1`, `@playwright/test@1.64.1`, `@types/node@22.20.5`,
  `@types/three@0.186.0`, `@vitest/coverage-v8@5.0.5`, `eslint@10.13.0`, `prettier@3.9.10`,
  `typescript@6.0.3`, `typescript-eslint@8.72.0`, `vite@8.3.4`, `vitest@5.0.5`.
- `package.json` has `"private": true`, `"type": "module"` and
  `"engines": {"node": "^22.13.0 || ^24.0.0 || >=26.0.0"}`.
- If an exact version cannot be installed, use the newest patch of the same minor version and log it
  in `docs/decisions.md`. Never use TypeScript 7.x: typescript-eslint 8.72 supports only TypeScript
  below 6.1.

## §2.5 npm scripts

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

## §2.6 Vite config (`vite.config.ts`)

- `mode` is `production` or `e2e`; `debug = mode !== 'production'`.
- `define: { __GB_DEBUG__: JSON.stringify(debug), __GB_VERSION__: JSON.stringify(<package.json version>) }`.
- `server` and `preview` use `host: '127.0.0.1'`, ports 5317 and 4317, `strictPort: true`.
- `build.outDir` is `dist-e2e` in e2e mode, otherwise `dist`. `build.sourcemap: debug`.
- `build.rolldownOptions.input`: `index.html`, plus `gallery.html` only when `debug`. Build the paths
  with `fileURLToPath(new URL('./x.html', import.meta.url))`.
- A build-only plugin (`apply: 'build'`) adds this CSP meta tag through `transformIndexHtml`
  (`injectTo: 'head-prepend'`): `default-src 'self'; script-src 'self'; style-src 'self'; img-src
  'self' data: blob:; connect-src 'self'; object-src 'none'; base-uri 'none'; form-action 'none'`.
