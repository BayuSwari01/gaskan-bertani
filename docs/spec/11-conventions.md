# §11 Conventions and Definition of Done

## §11.1 Code

- Strict TypeScript; no `any`, no `as unknown as`, no non-null `!` in `src`.
- `import type` for types; no enums, namespaces or parameter properties.
- Files ≤ 300 lines; one model family per file. Comments in English.
- No dependencies beyond §2.4. Core returns `Result` values and throws only on programmer errors.

## §11.2 Tests and the sprint Definition of Done

- Every task adds or updates tests. Never delete, skip or weaken a test to make it pass.
- A sprint is done when: all its tasks and Fix items are checked; `npm run check:all` is green inside
  the sandbox; from Sprint 4, `npm run test:coverage` meets the thresholds; `coder-report.md` lists
  the commands run, the tests added and the screenshots produced; decisions are appended to
  `docs/decisions.md`; nothing was written outside the project and no server is left running.

## §11.3 Project Definition of Done (the reviewer then writes DONE.md)

- All sprints approved.
- The full §1 loop is playable through the UI in both the desktop and mobile projects: crops with
  withering, trees, animals with bottles, well, mill, fertilizer, drag-to-act, levels and the bonus.
- Every model appears in the gallery and in the game.
- The persistence, offline-progress and fuzz suites are green; the §6.11 budgets are green; the §10.8
  security checklist is clean.
