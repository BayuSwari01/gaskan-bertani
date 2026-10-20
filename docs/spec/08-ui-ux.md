# §8 UI/UX

## §8.1 Layout

- HUD top-left: level badge and XP bar ("Lv 3 · 45/100 XP": XP inside the level / XP needed for the
  next level; "MAX" at level 20).
- HUD top-right: koin; air "12/30" with a "+1 02:13" countdown below the cap; pakan "8/50"; pupuk;
  botol.
- Toolbar at the bottom (full width on mobile, scrollable when narrow): Toko, Gudang, Bajak, Pupuk,
  Pindah, Jual, Pengaturan. The active tool is highlighted.
- Mode chip at the top centre whenever the mode is not normal: "Mode Tanam: Tomat", "Mode Bajak",
  "Mode Pupuk", "Mode Pindah", "Mode Jual", "Pasang: Jeruk", with a "Batal" button.
- Panels: one at a time; on desktop a centred modal up to 520 px wide; at ≤ 600 px a bottom sheet up
  to 70dvh, scrollable. Close with "Tutup", Esc or the backdrop.
- Respect safe-area insets; use `100dvh`; touch targets ≥ 44 px; font stack
  `system-ui, "Segoe UI", Roboto, "Noto Sans", sans-serif`.

## §8.2 Tapping objects in normal mode

| Target | Result |
|---|---|
| Empty plot | Seed Picker; choosing a crop plants this plot and enters seed mode with that crop |
| Seeded plot | water |
| Growing plot | object info |
| Ripe plot | harvest |
| Withered plot | Withered dialog: "Bersihkan" (clearPlot) or "Pulihkan (N koin)" (revive) |
| Tree needing water | water |
| Tree growing | object info |
| Tree ready | harvest |
| Hungry baby | feed (Botol Susu) |
| Hungry adult | feed (pakan) |
| Animal in cooldown | object info |
| Sumur full | collectWell |
| Sumur refilling | well panel |
| Gudang | Gudang panel |
| Penggilingan | mill panel |
| Rumah | object info |
| Empty tile | close the panel and deselect |

Desktop only: hovering an object shows a tooltip with its name, status text and time left, refreshed at
1 Hz.

## §8.3 Modes and drag-to-act

- **seed** (from the Seed Picker or the Toko's Benih tab): tapping or dragging over empty plots plants
  the active crop; other targets behave as in normal mode. Exits with "Batal", Esc, or when koin run out
  (toast).
- **plow** (Bajak): a plot ghost follows the hover (mouse) or the last tap (touch); tapping or
  dragging over free tiles plows. It stays active after `LIMIT_REACHED` (toast) until "Batal" or Esc.
- **fertilize** (Pupuk): tapping or dragging over plots and trees fertilizes them; exits when pupuk
  reaches 0 (toast), with "Batal" or Esc.
- **move** (Pindah, or Pindahkan in object info): the first tap picks any object (buildings
  included); its ghost follows; tapping a destination moves it; Esc or "Batal" cancels.
- **sell** (Jual): tapping an object opens a confirmation ("Jual Jeruk? Kamu mendapat 52 koin." or,
  for plots, "Hapus petak ini? Tanaman di atasnya akan hilang."); buildings show the `NOT_REMOVABLE`
  toast.
- **place** (Toko → Pohon or Hewan): ghost plus footprint quad, green when the spot is valid and
  affordable, otherwise red with the reason. Mouse: the ghost follows the hover and a click places.
  Touch: a tap moves the ghost and "Pasang" (or tapping the same tile again) places. One item per
  purchase; the mode ends after placing; Esc or "Batal" cancels without cost.
- **Drag-to-act** ("drag massal", FarmVille 2). A stroke starts only when the pointer goes down on a
  valid target for the mode's paint action:

| Mode | Paint action | Valid targets |
|---|---|---|
| normal | water | seeded plots (the stroke started on a seeded plot) |
| normal | harvest | ripe plots (the stroke started on a ripe plot) |
| seed | plant | empty plots |
| plow | plow | free tiles |
| fertilize | fertilize | plots and trees that can be fertilized |

  Each target is acted on at most once per stroke; the camera does not move during a stroke; other
  targets are skipped silently; the first failure for a missing resource (koin, air, pupuk) ends the
  stroke with one toast. Right/middle mouse drags and two-finger touches always pan and zoom.

## §8.4 Panels and modals

- **Seed Picker** (`seed`): every crop with icon, cost, grow time and product; locked crops are
  disabled and show "Level N".
- **Toko** (`shop`): tabs Benih (crops → seed mode), Pohon and Hewan (→ place mode), Perlengkapan
  (Botol Susu ×1, ×5, ×10). Each item shows its icon, name, price, key stats, lock level and
  affordability.
- **Gudang** (`storage`): goods with icon, count and price; quantity 1 / 5 / Semua; "Jual" with the
  total; a summary of pakan, pupuk and botol.
- **Penggilingan** (`mill`): millable goods the player owns with their ratio ("1 Jagung → 4 Pakan");
  quantity 1 / 5 / Maks; "Giling"; a pakan bar "x/50".
- **Sumur** (`well`): status ("Penuh" or the refill time), "Ambil Air (+10)", and a line about natural
  regeneration ("+1 air setiap 3 menit sampai 30").
- **Object info** (`objectInfo`): name (baby names for babies), status text, time left, progress bar,
  a fertilized badge, and the actions Pindahkan, Jual or Hapus (not for buildings), and Beri Pupuk
  (when allowed).
- **Pengaturan** (`settings`): Suara (on/off), Kurangi Animasi, Mulai Ulang (with confirmation), the
  version.
- Modals: confirm, level-up (queued), withered.

## §8.5 Feedback

- Floating text near the object, from events: "+1 Tomat", "+1 XP", "−1 Air", "−10 koin",
  "+1 Pupuk!", "+10 Air". It rises 40 px over 1.2 s; under e2e it is static for 2 s.
- Toasts: at most 3 at a time, 3 s each, `aria-live`.
- Level-up modal: "Naik Level N!", "+X koin", "+Y air", the unlocked items, and the line "Semua
  tanaman yang sudah disiram langsung siap panen!" when `levelBonus` ripened or revived anything.
  Several level-ups queue.
- Aid banner: "Bantuan Desa" with the button "Ambil 50 koin" while aid is available.
- Banners (Sprint 16): "Progres tidak dapat disimpan" when storage is unavailable; a read-only banner
  for a save from a newer version.
- Onboarding hints (Sprint 16), the FarmVille 2 tutorial order: "Panen 4 tomat yang sudah matang.
  Klik tomatnya!" → "Siram 4 tomat yang baru ditanam." → "Tanam benih di petak yang kosong." → "Jual
  tomat di Gudang untuk mendapat koin." → "Pilih Bajak, lalu klik rumput untuk membuat petak baru." →
  "Hebat! Selamat bertani!"

## §8.6 Strings, glossary and status texts

- All UI text lives in `src/ui/strings.id.ts`; display names live in the catalog.
- Vocabulary: petak, lahan, benih, bibit, siram, panen, layu, bersihkan, pulihkan, pupuk, bajak,
  gudang, sumur, air, pakan, penggilingan pakan, botol susu, koin, XP, level, tumbuh, siap panen,
  lapar, kenyang, beri pakan, beri susu, ambil air, jual, giling, pindahkan, hapus, pasang, batal,
  tutup, toko, dewasa, anak.
- Durations: "45d", "4m 05d", "1j 05m", "1h 02j" (h = hari, from 24 hours). Numbers use
  `Intl.NumberFormat('id-ID')`, so 1250 shows as "1.250".
- Status texts (keys used by `objectInfo` and the tooltip):

| Key | Text |
|---|---|
| plot.empty | "Petak kosong" |
| plot.seeded | "Perlu disiram" |
| plot.growing | "Tumbuh · siap dalam {time}" |
| plot.ripe | "Siap panen · layu dalam {time}" |
| plot.withered | "Layu" |
| tree.needsWater | "Perlu disiram ({n} air)" |
| tree.growing | "Berbuah dalam {time}" |
| tree.ready | "Siap panen" |
| baby.hungry | "Minta susu ({taken}/{need} botol)" |
| baby.cooldown | "Kenyang · minta susu lagi dalam {time}" |
| adult.hungry | "Lapar · butuh {n} pakan" |
| adult.cooldown | "Kenyang · lapar lagi dalam {time}" |
| sumur.full | "Penuh · ambil 10 air" |
| sumur.refilling | "Terisi penuh dalam {time}" |
| building.idle | "Bangunan" |

## §8.7 Error messages

| Code | Message |
|---|---|
| NOT_FOUND | "Objek tidak ditemukan." |
| WRONG_TARGET | "Aksi ini tidak bisa dilakukan di sini." |
| OUT_OF_BOUNDS | "Tidak bisa ditempatkan di luar lahan." |
| OCCUPIED | "Tempat ini sudah terisi." |
| LIMIT_REACHED (plots) | "Batas petak tercapai ({limit}). Naik level untuk menambah petak." |
| LIMIT_REACHED (animals) | "Jumlah {name} sudah maksimal ({limit})." |
| LIMIT_REACHED (objects, goods, coins, bottles) | "Batas tercapai." |
| NOT_ENOUGH_COINS | "Koin tidak cukup." |
| NOT_ENOUGH_WATER | "Air tidak cukup. Tunggu air terisi atau ambil dari Sumur." |
| NOT_ENOUGH_FEED | "Pakan tidak cukup. Giling hasil panen di Penggilingan Pakan." |
| NOT_ENOUGH_BOTTLES | "Botol Susu habis. Beli di Toko." |
| NOT_ENOUGH_FERTILIZER | "Pupuk habis. Beri makan hewan dewasa untuk mendapat pupuk." |
| NOT_ENOUGH_GOODS | "Barang di gudang tidak cukup." |
| FEED_FULL | "Tempat pakan sudah penuh." |
| LEVEL_LOCKED | "Terbuka di Level {level}." |
| PLOT_EMPTY | "Petak masih kosong. Tanam benih dulu." |
| PLOT_NOT_EMPTY | "Petak ini sudah ditanami." |
| ALREADY_WATERED | "Sudah disiram." |
| ALREADY_FERTILIZED | "Sudah dipupuk." |
| NOT_READY | "Belum siap. {time} lagi." |
| WITHERED | "Tanaman sudah layu." |
| NOT_WITHERED | "Tanaman ini tidak layu." |
| NOT_HUNGRY | "Hewan ini masih kenyang." |
| NOT_MILLABLE | "Barang ini tidak bisa digiling." |
| INVALID_QTY | "Jumlah tidak valid." |
| NOT_REMOVABLE | "Bangunan ini tidak bisa dijual." |
| AID_UNAVAILABLE | "Bantuan belum tersedia." |
| UNSUPPORTED | "Aksi tidak dikenal." |

## §8.8 Test ids and accessibility

- HUD: `hud-level`, `hud-xp`, `hud-coins`, `hud-water`, `hud-water-timer`, `hud-feed`,
  `hud-fertilizer`, `hud-bottles`.
- Toolbar and mode: `btn-shop`, `btn-storage`, `btn-plow`, `btn-fertilize`, `btn-move`,
  `btn-sell-tool`, `btn-settings`, `mode-chip`, `btn-mode-cancel`.
- Panels: `panel-<PanelId>`, `btn-close`.
- Seed Picker and Toko: `seed-<cropId>`, `shop-tab-<benih|pohon|hewan|perlengkapan>`,
  `shop-item-<crop|tree|animal>-<id>`, `buy-bottles-<1|5|10>`.
- Gudang: `goods-row-<goodId>`, `sell-qty-1`, `sell-qty-5`, `sell-qty-all`, `btn-sell`.
- Penggilingan: `mill-row-<goodId>`, `mill-qty-1`, `mill-qty-5`, `mill-qty-max`, `btn-mill-go`.
- Sumur: `btn-well-collect`.
- Placement: `place-bar`, `btn-place-confirm`, `btn-place-cancel`.
- Feedback and modals: `toast`, `float-text`, `tooltip`, `modal-levelup`, `modal-confirm`,
  `btn-confirm-yes`, `btn-confirm-no`, `modal-withered`, `btn-withered-clear`, `btn-withered-revive`.
- Object info: `info-name`, `info-status`, `info-progress`, `btn-move`, `btn-remove`,
  `btn-info-fertilize`.
- Other: `aid-banner`, `btn-aid`, `banner-storage`, `overlay-tab`, `overlay-gl`, `fatal`, `hint`,
  `settings-sound`, `settings-reduced-motion`, `btn-reset`.
- Use real `<button>` elements with accessible names, visible focus, and `aria-live` for toasts.
