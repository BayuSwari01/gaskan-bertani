# Decisions

Append-only log. Add one dated line per decision at the end; never edit or remove old lines.
Format: `- YYYY-MM-DD · decision — reason (who)`.

- 2026-10-20 · Scope is FarmVille 2's core loop only: crops on plots, fruit trees, and three animals (ayam, kambing, sapi), with 6 crops and 5 trees — owner request (owner)
- 2026-10-20 · Three.js low-poly 3D with a fixed isometric orthographic camera and procedural models, instead of a 2D engine — owner choice (owner)
- 2026-10-20 · Offline single player with saves in localStorage and timestamp-based timers — owner choice; keeps the scope small, and a backend can be added later because core is pure (owner)
- 2026-10-20 · Local Indonesian theme: padi, jagung, cabai, tomat, bawang merah, singkong; jeruk, mangga, rambutan, kelapa, durian — owner choice (owner)
- 2026-10-20 · In-game text in Bahasa Indonesia; code, comments, tests and docs in English — owner choice (owner)
- 2026-10-20 · Baby animals are raised with Botol Susu bought with koin — FarmVille 2 needed friends or real money; owner choice (owner)
- 2026-10-20 · Original FarmVille 2 durations (crops 1 minute to 24 hours, trees 2–12 hours, cow 3 hours) — owner choice (owner)
- 2026-10-20 · Include drag-to-act, fertilizer, withering and the level-up bonus — owner choice (owner)
- 2026-10-20 · Withering windows are fixed per crop at 2–24 hours, and revive costs 2× the seed cost — FarmVille 2 data is disputed; this replaces the premium Unwither item (planner)
- 2026-10-20 · Feeding an adult animal produces its good immediately (no separate collect step) — as in FarmVille 2 (planner)
- 2026-10-20 · The Gudang combines FarmVille 2's inventory and Market Stand, with no storage cap — FarmVille 2 documents no cap; one panel is simpler (planner)
- 2026-10-20 · Plowing is free and the plot limit rises with the level — FarmVille 2 raised it through farm expansions, which are out of scope (planner)
- 2026-10-20 · One Sumur only; no extra wells for sale — keeps the building set small (planner)
- 2026-10-20 · "Bantuan Desa" (50 koin, 10-minute cooldown) guards against a soft lock — a single player has no neighbors to help (planner)
- 2026-10-20 · Exact dependency versions pinned, TypeScript 6.0.3 instead of 7.x — typescript-eslint 8.72 supports only TypeScript below 6.1 (planner)
- 2026-10-20 · Headless WebGL through SwiftShader flag profiles, with npm cache, temp files and browsers inside the project — the agents run sandboxed on a GPU-less server (planner)
- 2026-10-20 · The specification lives in `docs/spec/` (one file per section) and `todo.md` holds only the sprints — the my-ai-agent convention: docs hold the spec, todo holds the plan (owner)
