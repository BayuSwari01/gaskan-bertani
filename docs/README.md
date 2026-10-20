# Gaskan Bertani documentation

Gaskan Bertani is a single-player browser game that clones the core loop of FarmVille 2 (crops on
plots, fruit trees, animals) in Three.js low-poly 3D. All in-game text is Bahasa Indonesia; code,
comments, tests and these documents are English.

This folder is the specification. `todo.md` in the project root is the work plan: sprints of checkbox
tasks that refer to the specification as §N.

## Reading order

1. `game-design.md`: what the game is, why each mechanic exists, worked examples with real numbers,
   and how a play session should feel. Read it first: it explains the intent behind the rules.
2. `visual-guide.md`: what the player sees: camera orientation, the starting farm, screen and panel
   mockups, the look of every object state, and the feedback choreography.
3. `glossary.md`: the Indonesian words used in the game, what they mean, and their code ids.
4. `spec/`: the normative specification, §1–§12 (map below).
5. `research-farmville2.md`: background on how FarmVille 2 works and what this clone copies, adapts
   or skips.
6. `decisions.md`: the append-only decision log.

## Specification map

§N lives in `spec/NN-*.md`; for example §4.3 is in `spec/04-game-rules.md`.

| § | File | Contents |
|---|---|---|
| §1 | `spec/01-product-scope.md` | the game, content, supporting systems, platform, out of scope |
| §2 | `spec/02-environment.md` | owner prerequisites, sandbox rules, config files, dependencies, npm scripts, Vite config |
| §3 | `spec/03-architecture.md` | folder layout, layers, enforcement, data flow, core types, store, time model, worked examples |
| §4 | `spec/04-game-rules.md` | grid, XP and levels, crops, trees, animals, water, well, goods, commands, invariants, state diagrams |
| §5 | `spec/05-game-data.md` | every balance number: economy, levels, crops, trees, animals, goods, buildings, starting farm |
| §6 | `spec/06-rendering.md` | renderer, camera, coordinates, input, picking, model kit, layers, lighting, animation, budgets |
| §7 | `spec/07-art-direction.md` | style, palette, model specs, scenery, icons, visual review checklist |
| §8 | `spec/08-ui-ux.md` | layout, tapping, modes and drag-to-act, panels, feedback, strings, errors, test ids |
| §9 | `spec/09-persistence.md` | storage keys, save envelope, validation, migrations, load and write, tab guard |
| §10 | `spec/10-testing.md` | test layers, URL flags, debug API, Playwright, E2E conventions, fixtures, screenshots, security |
| §11 | `spec/11-conventions.md` | code conventions, sprint and project Definition of Done |
| §12 | `spec/12-pitfalls.md` | known pitfalls to read before coding |

## Rules for these documents

- `spec/` is the contract between sprints and wins over everything else. Code follows the spec; never
  rewrite the spec to match code. Change it only when a task or a `### Fix` item asks for it, or when
  an implementation must deviate; then update the affected lines in the same commit and append the
  reason to `decisions.md`.
- `game-design.md`, `visual-guide.md`, `glossary.md` and `research-farmville2.md` explain intent and
  never override `spec/`. Their numbers are examples derived from §5; if §5 changes, §5 wins.
- Mockup text in `visual-guide.md` is the intended Indonesian copy for texts that §8 does not list.
- `decisions.md` is append-only: add dated lines at the end and never edit old ones.
- Balance numbers live in §5 and in `src/core/data/catalog.ts`, and the two stay identical.
- Never put checkboxes in `docs/`; tasks belong in `todo.md`.
