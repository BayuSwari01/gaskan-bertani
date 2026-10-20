# §12 Known pitfalls (read before coding)

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
