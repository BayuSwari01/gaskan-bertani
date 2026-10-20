# §9 Persistence

## §9.1 Keys

`gaskan-bertani:save`, `gaskan-bertani:backup`, `gaskan-bertani:corrupt` (quarantine, ≤ 200 KB),
`gaskan-bertani:settings` (`{ muted, reducedMotion, hintsStep, hintsDone }`, whitelist-validated). Only
`app/persistence.ts` and `app/settings.ts` touch localStorage, always through `StorageLike` and inside
try/catch.

## §9.2 Envelope

`{"app":"gaskan-bertani","version":1,"savedAt":<ms>,"state":GameState}`. Payloads over 1 MB are rejected
before parsing.

## §9.3 Validation (`validateSave(raw: unknown): Result<GameState, SaveError>`)

- `SaveError`: `PARSE_ERROR`, `TOO_LARGE`, `NOT_OBJECT`, `BAD_APP`, `UNSUPPORTED_VERSION`,
  `FUTURE_VERSION`, `INVALID_SCHEMA`, `INVARIANT_VIOLATION`.
- Build a fresh object field by field from a whitelist. Never spread or `Object.assign` untrusted data.
- Check types, finiteness and integer-ness; check the id regex before using an id as a key; only known
  definition ids; then every §4.11 invariant; at most `MAX_OBJECTS` objects.

## §9.4 Migrations

- A registry `{ [fromVersion]: (raw) => raw }`. Version 0 is a synthetic legacy format that exists only
  to exercise the pipeline: identical to v1 except `money` instead of `coins` and no `aidClaimedAt`.
- After Sprint 5, any change to the shape of `GameState` requires: bumping `SAVE_VERSION`, adding a
  migration, and adding a golden file `src/core/save/__fixtures__/vN-*.json`. Old golden files are
  never edited.

## §9.5 Load and write

- Load order: main → backup → new game. Outcomes: `new`, `loaded`, `recoveredFromBackup`,
  `resetCorrupt`, `futureVersion`, `storageUnavailable`.
  - A corrupt main save is copied to the quarantine key.
  - A future-version save is left untouched; the game runs in memory, read-only for storage, with a
    banner.
  - If storage is unavailable, the game runs in memory with the banner "Progres tidak dapat disimpan".
- Write: debounced 400 ms after each successful dispatch, flushed on `visibilitychange → hidden` and
  `pagehide`. On the first save of a session, copy a valid existing main save to the backup key.
- Quota exceeded: remove the quarantine and retry once; if it still fails, toast "Gagal menyimpan".

## §9.6 Tab guard (Sprint 16)

- `BroadcastChannel('gaskan-bertani')`; on boot a tab posts `hello`. An older tab that receives `hello`
  becomes inactive (no input, no saving) and shows the overlay "Gaskan Bertani sedang dibuka di tab
  lain." with the button "Main di sini", which reloads from storage and posts `hello`.
- Off under e2e unless `tabguard=1`.
