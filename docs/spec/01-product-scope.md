# §1 Product scope

- **Game.** "Gaskan Bertani", single-player and offline in the browser. It clones the FarmVille 2
  core loop and nothing else:
  - Crops on plots: plow a plot (free) → plant a seed (koin) → water it (1 air) → it grows → ripe →
    harvest (goods + XP). A ripe crop that is left too long withers.
  - Fruit trees: buy a sapling → water it (N air) → it grows → fruit ready → harvest → water again.
  - Animals: buy a baby → give it Botol Susu (milk bottles) until it is adult → an adult gets hungry →
    feed it pakan → its product pops out immediately (+XP, chance of pupuk) → cooldown → hungry again.
- **Content** (ids in parentheses):
  - Crops (6): Tomat (`tomat`), Padi (`padi`, its good is Gabah `gabah`), Cabai (`cabai`),
    Jagung (`jagung`), Singkong (`singkong`), Bawang Merah (`bawang_merah`).
  - Trees (5): Jeruk (`jeruk`), Mangga (`mangga`), Rambutan (`rambutan`), Kelapa (`kelapa`),
    Durian (`durian`).
  - Animals (3): Ayam (`ayam`, baby "Anak Ayam", product Telur `telur`), Kambing (`kambing`, baby
    "Cempe", product Susu Kambing `susu_kambing`), Sapi (`sapi`, baby "Pedet", product Susu Sapi
    `susu_sapi`).
- **Supporting systems** (FarmVille 2 mechanics, rules in §4): koin (coins, earned only by selling
  goods); XP and levels 1–20 with unlocks and a level-up bonus; air (water) that regenerates plus a
  Sumur (well); pakan (feed) made from goods in the Penggilingan Pakan (feed mill); pupuk
  (fertilizer); Botol Susu bought with koin; the Gudang (barn: inventory and selling); drag-to-act over
  many plots; withering; "Bantuan Desa" (aid that prevents a soft lock).
- **Platform.** Desktop and mobile browsers at or above Vite 8's baseline (Chrome/Edge 111,
  Firefox 114, Safari 16.4). WebGL2 is required.
- **Look.** Low-poly 3D with flat shading and a bright cartoon palette, under a fixed isometric
  orthographic camera that never rotates. Every model is built in code from primitives; there are no
  model, texture, audio or font files.
- **Out of scope:** farm expansion, crafting/kitchen, mastery, groves, prized animals,
  neighbors/social, quests, premium currency, object rotation, store-to-inventory, backend, network,
  accounts, analytics, service worker, music.
