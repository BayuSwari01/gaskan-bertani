# §6 Rendering

## §6.1 Renderer

- `WebGLRenderer({ canvas, antialias: !e2e && devicePixelRatio < 2, powerPreference: 'high-performance' })`.
- `setPixelRatio(min(devicePixelRatio, 2))` (1 under e2e); size with a `ResizeObserver` and
  `setSize(w, h, false)`.
- Default `outputColorSpace` (sRGB), `NoToneMapping`, shadow maps off, `scene.background = #BFE9FF`.
- Never set `failIfMajorPerformanceCaveat`. Construction is wrapped in try/catch and falls back to the
  §6.10 message.

## §6.2 Camera (`IsoCameraRig`)

- `OrthographicCamera`, yaw 45°, pitch `atan(1/√2)` (35.264°). Position `target + (0.577, 0.577,
  0.577) × 60`, looking at the target; near 0.1, far 200.
- `VIEW_SIZE = 14` world units across the shorter canvas side at zoom 1: `half = 7`; landscape
  `top = half`, `right = half × aspect`; portrait `right = half`, `top = half / aspect`.
- Zoom range [0.4, 2.5], default 1. The target is clamped to x, z in [−2, 26]; the initial target is
  (11.5, 0, 11.5), the centre of the starting plots.
- Pixels per world unit: `ppu = shortSidePx / VIEW_SIZE × zoom`. The camera never rotates.

## §6.3 Coordinates

- An object's world position is its footprint centre `(x + w/2, 0, y + h/2)`.
- World to grid: `floor(worldX)`, `floor(worldZ)`.
- Screen coordinates are always CSS px relative to the viewport (`canvas.getBoundingClientRect()`),
  never drawing-buffer px.

## §6.4 Input

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

## §6.5 Picking (in this order)

1. Bubbles: within `0.6 × 44 px` of a bubble's screen centre.
2. `Raycaster.setFromCamera` against object meshes (`userData.objectId`) and invisible animal proxy
   boxes (footprint × 0.8 height); the nearest hit wins. Call `scene.updateMatrixWorld()` first.
3. The ground plane y = 0, giving a tile inside the farm (and the object on that tile, if any).
4. Otherwise none.

Raycasting what is visible means clicking a tree's canopy selects the tree. Ghost meshes, overlays and
scenery are never pickable.

## §6.6 Model kit and cache

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

## §6.7 Layers, heights and render order

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

## §6.8 Lighting

- `HemisphereLight(#FFFFFF, #6B8E4E, 1.6)` plus `DirectionalLight(#FFF4E0, 2.2)` at target +
  (−6, 12, 8). Tune in the gallery. No shadow maps; blob shadows only.

## §6.9 Animation (Sprint 14)

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

## §6.10 Loop and lifecycle

- Render on demand (`requestRender()`). Render continuously only while a gesture, inertia, tween or
  wander is active.
- When `document.hidden`: stop the loop and the 1 Hz tick and flush the save. When visible again:
  `store.tick()`, resync, render.
- `webglcontextlost`: `preventDefault`, stop, show the overlay "Memulihkan grafis…". On
  `webglcontextrestored`: set the atlas `needsUpdate`, resume, hide the overlay.
- If WebGL2 is missing or the renderer throws, `#fatal` shows: "Browser ini tidak mendukung WebGL2.
  Aktifkan akselerasi grafis atau gunakan Chrome, Firefox, atau Safari terbaru."

## §6.11 Performance budgets

- Triangles per model: plot + crop 500; tree 900; fruit overlay 300; `fx:fertilized` 100; animal 600;
  building 2000; decoration 300; merged scenery 40k.
- Draw calls ≤ 2 × objects + 20.
- `stress` fixture: ≤ 320 draw calls, ≤ 220k triangles, `profileSync()` ≤ 8 ms.
- Memory: geometries ≤ (model keys in use) + 15, textures ≤ 4, both stable over 50 reset cycles.
- Production JS ≤ 1.3 MB minified.
- Real hardware (checked by a human with `?perf=1`, never asserted on SwiftShader): 60 fps for the
  stress scene on a desktop integrated GPU at 1080p, ≥ 45 fps with 150 objects on a mid-range phone.
