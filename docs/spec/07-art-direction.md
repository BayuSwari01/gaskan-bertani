# §7 Art direction

## §7.1 Style

- Chunky low-poly primitives, flat shading, bright saturated colours. Cylinders and cones use 5–8
  segments; icosahedra use detail 0–1.
- 1 world unit = 1 tile. Models face +z, stand at y = 0 and are centred on their footprint.
- Small hash-seeded variation inside a model is allowed; every model is deterministic per key.

## §7.2 Palette (sRGB hex)

| Group | Colours |
|---|---|
| Sky and ground | sky #BFE9FF; grass #8CC63F; grassAlt #7FB935; grassOuter #6FA832 |
| Soil | soilDry #C9965F; soilWet #8A5A3A; furrow #6E4630; seed #4E342E |
| Plants | leaf #4CAF50; leafDark #2E7D32; leafLight #8BD45A; stem #6B8E23; trunk #8B5A2B; trunkDark #6D4C41 |
| Crop produce | padiGold #E8C547; cornYellow #FFD54F; cornHusk #A5D66B; tassel #B08D57; chiliRed #E53935; tomatoRed #F44336; unripeGreen #9CCC65; shallot #A23E5C; cassavaSkin #8D6E63; cassavaStem #A1887F; flowerWhite #FFFDE7 |
| Wither and fertilizer | wither #8D6E4A; witherDark #6D5A3F; sparkle #FFE082; sparkleCore #FFF8E1 |
| Fruit | mango #FFB300; mangoBlush #FF7043; rambutan #D32F2F; durian #9E9D24; durianSpike #6B6B1F; coconut #7CB342; coconutBrown #795548; frond #43A047; orange #FB8C00 |
| Animals | white #FAFAFA; black #37474F; comb #E53935; beak #FFB300; chick #FFE082; pink #F8BBD0; horn #FFF3E0; goat #D2B48C; goatDark #8D6E63 |
| Buildings and props | woodLight #C19A6B; woodDark #8D6E63; barnRed #C0392B; roof #7B2E2E; trim #FFFFFF; stone #9E9E9E; stoneDark #757575; water #4FC3F7; metal #B0BEC5; sack #E6D3A3; path #D7B98E; houseWall #F5E6C8; houseRoof #D9534F; window #81D4FA; chimney #9E9E9E |
| Flowers | petalPink #F48FB1; petalYellow #FFEE58; petalPurple #BA68C8 |

## §7.3 Model specs (heights in world units)

- **Plot.** Raised bed 0.92 × 0.08 × 0.92 with three furrow ridges (0.86 × 0.03 × 0.14, `furrow`).
  `plot:empty` is dry soil; `plot:seeded` is dry soil with 9 seed dots; crop stages 1–4 sit on wet
  soil; `plot:withered` is dry soil with 6–9 stalks drooping 40–60° in `wither`/`witherDark`.
- **`fx:fertilized`.** 7 small gold octahedra (radius 0.03–0.05, `sparkle` and `sparkleCore`) at
  heights 0.12–0.3 over a 0.8 × 0.8 area; trees use it scaled ×2 around the trunk base.
- **Crop layout.** A 3×3 grid at ±0.28 for padi and bawang merah; a 2×2 grid at ±0.22 for the others.

| Crop | Stage 1 | Stage 2 | Stage 3 | Stage 4 (ripe) |
|---|---|---|---|---|
| padi (clumps of 4–5 three-segment blade cones, tilted ±12°) | h 0.10, leafLight | h 0.22 | h 0.38, pale panicles | h 0.45; padiGold upper blades and drooping golden panicles |
| jagung | 2-leaf sprout, h 0.12 | stalk h 0.35 with 4 angled flat leaves | h 0.7, tassel bud | h 0.95; tassel cones; 1–2 cornYellow cobs partly wrapped in cornHusk |
| cabai | sprout | leafDark bush h 0.18 | h 0.30, flowerWhite dots | h 0.38; 5–6 hanging chiliRed cones and 1–2 green ones |
| tomat (woodLight stake from stage 2) | sprout | bush h 0.22 | h 0.40, unripeGreen balls | h 0.48; 4–5 tomatoRed balls |
| bawang merah (tufts of 4–6 thin four-segment tubes) | h 0.08 | h 0.16 | h 0.24 | h 0.26 with yellowing tips; shallot bulbs half-exposed at soil level |
| singkong (cassavaStem stems topped by palmate clusters of 5–7 flat cones) | h 0.12 | h 0.35 | h 0.60 | h 0.70; 2–3 cassavaSkin tubers lying on the soil |

- **Trees** (2×2 footprint, canopy diameter ≤ 1.9). Stages 0–2 have heights 0.4 / 1.0 / 1.7; stage 0
  is a sapling (thin stem, 2–4 leaves, dark soil mound). `fruit-unripe` uses the same positions as
  `fruit-ripe` at 70% size in `unripeGreen` (kelapa: small `coconut` green).

| Tree | Stage 3 height | Shape | Ripe fruit |
|---|---|---|---|
| jeruk | 2.0 | compact round leafLight canopy | about 10 small `orange` balls |
| mangga | 2.6 | broad dome of 3–4 leafDark icosahedra | 8 oval `mango` fruits with `mangoBlush` |
| rambutan | 2.3 | dense round canopy | 6 clusters of 3–4 small `rambutan` balls with tiny spikes |
| kelapa | 3.2 | palm: curved trunk of 6 offset tapered segments in alternating tones; 7–8 drooping `frond` strips | 4–5 coconuts, mixed `coconut` and `coconutBrown` |
| durian | 3.0 | tall trunk, tiered narrow canopy | 4 large spiky `durian` fruits (spikes `durianSpike`) on low branches |

- **Animals**

| Animal | Adult | Baby |
|---|---|---|
| ayam (1×1) | h 0.45; white egg-shaped body; comb, wattle, beak; tail feathers; orange legs | Anak Ayam: yellow (`chick`) round body, h 0.2 |
| kambing (2×2) | h 0.75, length 0.8; `goat` box body; head with backward horns and a `goatDark` beard; four legs; upright tail | Cempe: 60% size, larger head, no horns |
| sapi (2×2) | h 1.0, length 1.2; white body with black patches that protrude 0.01 (no coplanar faces); pink muzzle and udder; `horn` horns; tufted tail | Pedet: 60% size, larger head, no horns or udder |

- **Buildings**

| Building | Description |
|---|---|
| rumah (3×3) | `houseWall` walls 2.4 × 1.4 × 2.0; `houseRoof` gable roof with 0.15 overhang; door; two `window` windows with `trim` frames; `chimney`; small porch with two posts; height ≤ 2.6 |
| gudang (3×3) | `barnRed` barn body 2.6 × 1.8 × 2.4; `roof` gable; `trim` trims; front double doors with an X; loft window |
| penggilingan (3×3) | `metal` silo (r 0.6, h 2.6) with a cone roof; small wooden shed; yellow hopper; `sack` sacks |
| sumur (2×2) | `stone` ring (r 0.45, h 0.45); two posts h 1.3; crank; small red gable roof; bucket. `full`: `water` disc at y 0.35; `empty`: `stoneDark` disc at y 0.1 |

- **Decorations:** pagar (fence: straight and corner), semak (bush ×3), batu (rock ×3), bunga
  (flowers ×3), border trees ×3 (broadleaf and conifer, no fruit), jalan (path stones).

## §7.4 Scenery

- A fence runs along the farm boundary. An 8-tile ring outside it holds seeded decoration placements on
  `grassOuter`, merged into one geometry with one draw call. Nothing is placed inside the farm.

## §7.5 Icons

- IconIds: `air`, `pakan`, `pupuk`, `botol`, `koin`, `xp`, `gembok` (lock), `jam` (clock), `layu`
  (wilted leaf), `ayam`, `kambing`, `sapi`, plus all 14 GoodIds (26 icons).
- Drawn with Canvas2D (paths, arcs, gradients) with a dark outline (#3E2723, about 6% of the size).
  No text and no emoji.
- Atlas of 6 × 5 cells of 128 px. The UI calls `drawIcon(ctx, id, size)` on small `<canvas>` elements.

## §7.6 Visual review checklist (the tester checks screenshots)

1. Each crop is distinguishable at stage 4, and stages visibly grow from 1 to 4.
2. Empty and seeded (dry) soil look different from watered (wet) soil.
3. Withered plots read as dead and brown; fertilized plots and trees show visible gold sparkles.
4. Tree species and their fruit are distinguishable; the palm reads as a palm; unripe (green) and ripe
   fruit are distinguishable.
5. Animal species and baby versus adult are distinguishable.
6. Buildings are recognizable and consistently scaled; the sumur shows full versus empty.
7. Icons are legible at 32 px and distinct from each other.
8. Faceted flat look; no missing faces, z-fighting or flicker.
9. Bubbles are legible at 44 px and keep the same size at every zoom; all UI text is Indonesian.
