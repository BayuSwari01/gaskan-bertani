# Visual guide

What the player sees. The normative rules are §6 (rendering), §7 (art) and §8 (UI); this guide shows
them as pictures. In the mockups, `[koin]`, `[air]` and similar names stand for icons drawn from the
Canvas2D atlas (§7.5): the game never uses emoji. Mockup text is the intended Indonesian copy for any
text that §8 does not list.

## 1. World orientation

The farm is a 24×24 grid. Tile `(x, y)` covers world `[x, x+1] × [y, y+1]` on the ground plane
(grid y is world z). The camera looks from the `(24, 24)` side, so on screen:

```
                         (0,0)  back corner, top of the screen
                          /\
                         /  \
          +y (world z)  /    \  +x
            goes       /      \  goes
          lower-left  /        \ lower-right
                     /          \
               (0,24)\          /(24,0)
                      \        /
                       \      /
                        \    /
                         \  /
                          \/
                       (24,24)  nearest to the viewer, bottom of the screen
```

The camera never rotates. Zoom is 0.4–2.5; at zoom 1 about 14 tiles fit across the shorter side of
the screen.

## 2. The starting farm (top view, one character per tile)

`R` Rumah, `G` Gudang, `P` Penggilingan Pakan, `S` Sumur, `T` ripe Tomat, `t` seeded Tomat (needs
water), `o` empty plot, `.` grass.

```
        x: 0         1         2
           012345678901234567890123
   y  0    ........................
      1    .RRR.GGG.PPP.SS.........
      2    .RRR.GGG.PPP.SS.........
      3    .RRR.GGG.PPP............
      4    ........................
      5    ........................
      6    ........................
      7    ........................
      8    ........................
      9    ........................
     10    ..........TTT...........
     11    ..........Ttt...........
     12    ..........tto...........
     13    ........................
     ...   (rows 13-23 are grass)
```

The camera starts centred on the plots, at world (11.5, 0, 11.5). Outside the 24×24 farm there is a
fence and an 8-tile ring of decorative scenery (bushes, rocks, flowers, border trees, path stones).

## 3. Desktop screen (1280 × 720)

```
+----------------------------------------------------------------------------------------------+
| (Lv 3) [=========--------] 45/100 XP          [koin] 1.250  [air] 12/30 +1 02:13  [pakan] 8/50 |
|                                                                  [pupuk] 2   [botol] 0         |
|                           +----------------------------------+                                |
|                           |  Mode Tanam: Tomat     [ Batal ]  |  <- mode chip, only in a mode  |
|                           +----------------------------------+                                |
|                                                                                                |
|                              isometric 3D farm (canvas)                                        |
|                    bubbles float above objects; hovering shows a tooltip                       |
|                                                                                                |
|  +--------------------------------------+                                                      |
|  | Panen 4 tomat yang sudah matang.     |  <- onboarding hint (Sprint 16)                       |
|  +--------------------------------------+                       [ Koin tidak cukup. ] <- toast |
+----------------------------------------------------------------------------------------------+
| [Toko] [Gudang] [Bajak] [Pupuk] [Pindah] [Jual]                                 [Pengaturan]   |
+----------------------------------------------------------------------------------------------+
```

The hover tooltip (mouse only) sits next to the cursor:

```
+------------------------------+
| Tomat                        |
| Tumbuh · siap dalam 42d      |
+------------------------------+
```

## 4. Mobile portrait (390 × 844)

```
+--------------------------------+
| (Lv 3) [======----] 45/100 XP  |
| [koin] 1.250  [air] 12/30 +1 2:13|
| [pakan] 8/50 [pupuk] 2 [botol] 0|
|                                |
|        3D farm (canvas)        |
|                                |
| +----------------------------+ |
| | Gudang              Tutup  | |  <- bottom sheet, at most 70dvh,
| | ...                        | |     scrolls inside
| +----------------------------+ |
| [Toko][Gudang][Bajak][Pupuk] > |  <- toolbar scrolls sideways
+--------------------------------+
```

## 5. Panels and modals

Seed Picker (opened by tapping an empty plot):

```
+------------------------------------------------+
| Pilih Benih                            [Tutup] |
|------------------------------------------------|
| [tomat]  Tomat      10 koin   1m 00d  -> Tomat |
| [gabah]  Padi       10 koin   4j 00m  -> Gabah |
| [cabai]  Cabai      15 koin   2m 00d  -> Cabai |
| [gembok] Jagung     Level 4             (off)  |
| [gembok] Singkong   Level 5             (off)  |
+------------------------------------------------+
```

Toko:

```
+--------------------------------------------------------+
| Toko                                           [Tutup] |
| [Benih] [Pohon] [Hewan] [Perlengkapan]                 |
|--------------------------------------------------------|
| [jeruk]   Jeruk     260 koin   3 air, 12j 00m  [Beli]  |
|           6 Jeruk setiap panen                         |
| [mangga]  Mangga    320 koin   Level 3          (off)  |
|--------------------------------------------------------|
| Perlengkapan tab:  Botol Susu  20 koin  [x1] [x5] [x10]|
+--------------------------------------------------------+
```

Gudang:

```
+--------------------------------------------------------+
| Gudang                                         [Tutup] |
| Pakan 8/50     Pupuk 2     Botol Susu 0                |
|--------------------------------------------------------|
| [tomat]  Tomat   x14   12 koin   <- selected           |
| [telur]  Telur   x3    60 koin                         |
|--------------------------------------------------------|
| Jumlah: [1] [5] [Semua]       Total: 168 koin  [Jual]  |
+--------------------------------------------------------+
```

Penggilingan Pakan:

```
+--------------------------------------------------------+
| Penggilingan Pakan                             [Tutup] |
| Pakan [=========-------------] 28/50                   |
|--------------------------------------------------------|
| [singkong] Singkong x2    1 Singkong -> 10 Pakan       |
| [tomat]    Tomat    x14   1 Tomat -> 1 Pakan           |
|--------------------------------------------------------|
| Jumlah: [1] [5] [Maks]    Hasil: +10 Pakan   [Giling]  |
+--------------------------------------------------------+
```

Sumur:

```
+--------------------------------------------------------+
| Sumur                                          [Tutup] |
| Penuh! Ambil 10 air.                 [Ambil Air (+10)] |
| +1 air setiap 3 menit sampai 30                        |
+--------------------------------------------------------+
```

Object info (a growing, fertilized tomato):

```
+--------------------------------------------------------+
| Tomat                                          [Tutup] |
| Tumbuh · siap dalam 42d                    (Dipupuk)   |
| [============--------]                                 |
| [Pindahkan]  [Hapus]                                   |
+--------------------------------------------------------+
```

Withered dialog:

```
+--------------------------------------------------------+
| Tanaman layu                                           |
| Cabai ini sudah layu dan tidak bisa dipanen.           |
| [Bersihkan (gratis)]            [Pulihkan (30 koin)]   |
+--------------------------------------------------------+
```

Sell confirmation (Jual tool on a tree):

```
+------------------------------------------+
| Jual Jeruk? Kamu mendapat 52 koin.       |
|                    [Ya]      [Tidak]     |
+------------------------------------------+
```

Level-up modal:

```
+--------------------------------------------------+
|                  Naik Level 3!                   |
|            +200 koin        +30 air              |
|   Terbuka:  [cabai] Cabai   [mangga] Mangga      |
|             [ayam] Ayam                          |
|   Semua tanaman yang sudah disiram langsung      |
|   siap panen!                                    |
|                    [ Lanjut ]                    |
+--------------------------------------------------+
```

## 6. What each object state looks like

| Object and state | Model key(s) | Bubble | Tooltip status |
|---|---|---|---|
| Plot, empty | `plot:empty` (dry soil) | – | "Petak kosong" |
| Plot, seeded | `plot:seeded` (dry soil, seed dots) | water drop | "Perlu disiram" |
| Plot, growing | `plot:<crop>:1`–`3` (wet soil) | – | "Tumbuh · siap dalam 3m 05d" |
| Plot, ripe | `plot:<crop>:4` (wet soil) | product icon | "Siap panen · layu dalam 1j 59m" |
| Plot, withered | `plot:withered` (dry soil, brown stalks) | wilted leaf (`layu`) | "Layu" |
| Plot, fertilized | base key + `fx:fertilized` (gold sparkles) | – | badge "Dipupuk" in object info |
| Tree, first cycle, needs water | `tree:<id>:0` (sapling) | water drop | "Perlu disiram (3 air)" |
| Tree, first cycle, growing | `tree:<id>:0` → `1` → `2` | – | "Berbuah dalam 2j 10m" |
| Tree, ready | `tree:<id>:3` + `tree:<id>:fruit-ripe` | fruit icon | "Siap panen" |
| Tree, later cycle, needs water | `tree:<id>:3` | water drop | "Perlu disiram (3 air)" |
| Tree, later cycle, growing | `tree:<id>:3` (+ `fruit-unripe` from 50%) | – | "Berbuah dalam 40m 00d" |
| Baby, hungry | `animal:<id>:baby` | bottle (`botol`) | "Minta susu (1/2 botol)" |
| Baby, cooldown | `animal:<id>:baby` | – | "Kenyang · minta susu lagi dalam 3m 10d" |
| Adult, hungry | `animal:<id>:adult` | pakan sack (`pakan`) | "Lapar · butuh 3 pakan" |
| Adult, cooldown | `animal:<id>:adult` | – | "Kenyang · lapar lagi dalam 4m 30d" |
| Sumur, full | `building:sumur:full` | water drop (`air`) | "Penuh · ambil 10 air" |
| Sumur, refilling | `building:sumur:empty` | – | "Terisi penuh dalam 3j 12m" |
| Rumah, Gudang, Penggilingan | `building:<id>` | – | "Bangunan" |

## 7. Recognition cues

A player must tell objects apart at a glance from the default zoom. The key cue of each model:

| Model | Silhouette | Colour cue |
|---|---|---|
| Tomat | small staked bushes | red balls |
| Padi | 3×3 thin grass clumps | gold drooping panicles |
| Cabai | low dark bushes | small hanging red cones |
| Jagung | tallest crop, upright stalks | yellow cobs, brown tassels |
| Singkong | tall thin stems with star-shaped leaf clusters | brown tubers on the soil |
| Bawang Merah | 3×3 thin upright tubes | purple-red bulbs at soil level |
| Jeruk | small round tree | many small orange balls |
| Mangga | broad dark dome | oval yellow-orange fruit with a red blush |
| Rambutan | dense round tree | red spiky clusters |
| Kelapa | the only palm: curved segmented trunk, drooping fronds | green and brown coconuts under the fronds |
| Durian | the tallest broadleaf, tiered canopy | large green spiky fruit low on the branches |
| Ayam / Anak Ayam | egg-shaped bird / tiny round ball | white with a red comb / yellow |
| Kambing / Cempe | box body, horns, beard / small, big head | tan and brown |
| Sapi / Pedet | the largest animal, boxy / small, big head | white with black patches |
| Rumah | small house with a chimney | cream walls, red roof |
| Gudang | the classic barn with an X on the doors | barn red and white trim |
| Penggilingan Pakan | tall metal silo with a shed | grey silo, yellow hopper, sacks |
| Sumur | round stone well with a little roof | blue water when full, dark when empty |

## 8. Feedback choreography

Harvesting a ripe plot in normal play (Sprint 14):

```
 t = 0 ms     click -> state updated, bubble removed, model swaps to plot:empty, HUD numbers update
 0-150 ms     the plot squashes 1 -> 1.15 -> 1
 0-600 ms     a Tomat icon arcs from the plot to the Gudang toolbar button, which pulses on arrival
 0-1200 ms    "+1 Tomat" and "+1 XP" rise 40 px and fade, stacked about 18 px apart
```

- Drag-to-act: each plot reacts the moment the stroke enters it, so a sweep looks like a chain of pops.
- Level-up: the modal opens after the triggering action's floating texts start; the crops ripened by
  the bonus pop (scale 0.7 -> 1) as their models change.
- Placing a tree or animal: the new object pops in; a baby animal starts wandering inside its footprint.
- Errors: a toast only; the scene does not change.
- Under `?e2e=1` there is no squash, fly, pop, bob or wander, and floating texts stay still for 2 s, so
  screenshots are stable.
