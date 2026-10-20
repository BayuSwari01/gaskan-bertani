# §5 Data tables (all values live in `src/core/data/catalog.ts`)

`catalogTypes.ts` defines `Catalog` with `economy` (§5.1), `levels` (§5.2: `{ level, xp, rewardCoins,
rewardWater }`), `crops` (§5.3), `trees` (§5.4), `animals` (§5.5), `goods` (§5.6), `buildings` (§5.7)
and `startingPlots` (§5.8). Records are keyed by id, every definition has `id` and `name` (display
name, Indonesian). Durations are in ms; "m" = minutes, "j" = jam (hours).

## §5.1 Economy

| Key | Value |
|---|---|
| startCoins | 150 |
| startWater / waterRegenCap / waterHardCap | 30 / 30 / 999 |
| waterRegenMs | 180000 (3 m) |
| wellWater / wellRefillMs | 10 / 14400000 (4 j) |
| startFeed / feedCap | 0 / 50 |
| startFertilizer / fertilizerCap | 2 / 99 |
| startBottles / bottleCap / bottlePrice | 0 / 999 / 20 |
| plotBase / plotPerLevel / plotMax | 12 / 4 / 80 |
| reviveMultiplier | 2 |
| removeRefundPercent | 20 |
| xpWaterPlot / xpFertilize / xpBottle | 1 / 1 / 1 |
| aidCoins / aidCooldownMs | 50 / 600000 (10 m) |

Constants in `constants.ts`: `FARM_SIZE = 24`, `MAX_OBJECTS = 300`, `COINS_MAX = 1_000_000_000`,
`XP_MAX = 1_000_000_000`, `GOOD_MAX = 999_999`, `SAVE_VERSION = 1`, `TIME_MIN`, `TIME_MAX`.

## §5.2 Levels (cumulative XP; unlocks are derived from each definition's `unlockLevel`)

| Level | XP | Reward koin | Reward air | Unlocks |
|---|---|---|---|---|
| 1 | 0 | 0 | 0 | tomat |
| 2 | 20 | 100 | 30 | padi, jeruk |
| 3 | 80 | 200 | 30 | cabai, mangga, ayam |
| 4 | 180 | 300 | 30 | jagung |
| 5 | 330 | 650 | 20 | singkong, rambutan, kambing |
| 6 | 580 | 750 | 10 | bawang_merah |
| 7 | 980 | 850 | 10 | kelapa |
| 8 | 1580 | 950 | 10 | sapi |
| 9 | 2430 | 1050 | 10 | – |
| 10 | 3530 | 1150 | 10 | durian |
| 11 | 4930 | 1250 | 10 | – |
| 12 | 6630 | 1400 | 10 | – |
| 13 | 8630 | 1550 | 10 | – |
| 14 | 11030 | 1700 | 10 | – |
| 15 | 13830 | 1900 | 10 | – |
| 16 | 17030 | 2000 | 15 | – |
| 17 | 20730 | 2100 | 15 | – |
| 18 | 24930 | 2200 | 15 | – |
| 19 | 29730 | 2300 | 15 | – |
| 20 | 35130 | 2400 | 15 | – |

## §5.3 Crops (each uses 1 air per watering; `yield` 1)

| id | name | unlockLevel | seedCost | growMs | witherMs | product | xpHarvest |
|---|---|---|---|---|---|---|---|
| tomat | Tomat | 1 | 10 | 60000 (1 m) | 7200000 (2 j) | tomat | 1 |
| padi | Padi | 2 | 10 | 14400000 (4 j) | 28800000 (8 j) | gabah | 2 |
| cabai | Cabai | 3 | 15 | 120000 (2 m) | 7200000 (2 j) | cabai | 1 |
| jagung | Jagung | 4 | 15 | 28800000 (8 j) | 43200000 (12 j) | jagung | 4 |
| singkong | Singkong | 5 | 18 | 86400000 (24 j) | 86400000 (24 j) | singkong | 13 |
| bawang_merah | Bawang Merah | 6 | 22 | 43200000 (12 j) | 64800000 (18 j) | bawang_merah | 7 |

## §5.4 Trees (2×2 footprint)

| id | name | unlockLevel | cost | waterPerCycle | cycleMs | fruit | fruitYield | xpWater | xpHarvest |
|---|---|---|---|---|---|---|---|---|---|
| jeruk | Jeruk | 2 | 260 | 3 | 43200000 (12 j) | jeruk | 6 | 1 | 4 |
| mangga | Mangga | 3 | 320 | 2 | 14400000 (4 j) | mangga | 4 | 1 | 2 |
| rambutan | Rambutan | 5 | 480 | 3 | 28800000 (8 j) | rambutan | 6 | 1 | 5 |
| kelapa | Kelapa | 7 | 620 | 3 | 7200000 (2 j) | kelapa | 4 | 1 | 4 |
| durian | Durian | 10 | 1800 | 4 | 43200000 (12 j) | durian | 4 | 1 | 10 |

## §5.5 Animals

| id | name / babyName | unlockLevel | cost | size | bottlesToAdult | bottleIntervalMs | feedPerMeal | cooldownMs | product × yield | xpFeed | fertilizerChance | maxOwned |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ayam | Ayam / Anak Ayam | 3 | 600 | 1×1 | 2 | 300000 (5 m) | 3 | 300000 (5 m) | telur × 1 | 2 | 0.2 | 10 |
| kambing | Kambing / Cempe | 5 | 2000 | 2×2 | 3 | 900000 (15 m) | 4 | 900000 (15 m) | susu_kambing × 1 | 3 | 0.3 | 6 |
| sapi | Sapi / Pedet | 8 | 6000 | 2×2 | 6 | 3600000 (1 j) | 8 | 10800000 (3 j) | susu_sapi × 2 | 7 | 0.5 | 4 |

## §5.6 Goods (sell price in koin; `millFeed` = pakan per item, absent = cannot be milled)

| id | name | price | millFeed |
|---|---|---|---|
| tomat | Tomat | 12 | 1 |
| gabah | Gabah | 14 | 2 |
| cabai | Cabai | 17 | 2 |
| jagung | Jagung | 20 | 4 |
| singkong | Singkong | 24 | 10 |
| bawang_merah | Bawang Merah | 30 | 6 |
| jeruk | Jeruk | 17 | 3 |
| mangga | Mangga | 18 | 2 |
| rambutan | Rambutan | 24 | 4 |
| kelapa | Kelapa | 32 | 3 |
| durian | Durian | 98 | 5 |
| telur | Telur | 60 | – |
| susu_kambing | Susu Kambing | 85 | – |
| susu_sapi | Susu Sapi | 85 | – |

## §5.7 Buildings (movable, never removable)

| id | name | start (x, y) | size | Tap action |
|---|---|---|---|---|
| rumah | Rumah | (1, 1) | 3×3 | object info (decorative farmhouse) |
| gudang | Gudang | (5, 1) | 3×3 | Gudang panel (inventory and selling) |
| penggilingan | Penggilingan Pakan | (9, 1) | 3×3 | mill panel |
| sumur | Sumur | (13, 1) | 2×2 | collect water when full, otherwise well panel |

## §5.8 Starting farm (`newGame(now, catalog, seed = 1)`, the FarmVille 2 tutorial farm)

- `coins 150`, `xp 0`, `water { amount: 30, updatedAt: now }`, `feed 0`, `fertilizer 2`,
  `bottles 0`, `goods {}`, `rng = seed >>> 0`, `aidClaimedAt null`, `createdAt = lastSeenAt = now`.
- The four buildings at their §5.7 positions, ids allocated in the order rumah, gudang, penggilingan,
  sumur; the sumur starts full (`collectedAt null`).
- `startingPlots`: nine plots on tiles x 10–12, y 10–12, allocated in row-major order (y, then x).
  The first four hold a ripe Tomat (`plantedAt = wateredAt = now − growMs`), the next four a seeded
  Tomat (`plantedAt = now`, `wateredAt = null`), the last one is empty.

## §5.9 Catalog validation (`validateCatalog`)

- Ids are unique and equal their keys; names are non-empty.
- `unlockLevel` in [1, 20]; costs, prices, yields and caps are positive integers; durations are
  positive integer multiples of 1000; `fertilizerChance` in [0, 1].
- Every product and fruit is a known good; `millFeed`, when present, is an integer ≥ 1.
- Every crop's product sells for more than its seed cost; `waterPerCycle ≤ waterRegenCap`;
  `feedPerMeal ≤ feedCap`; at least one level-1 crop has `seedCost ≤ startCoins`.
- The level table has 20 rows, starts at XP 0 and strictly increases; rewards are ≥ 0.
- Buildings and starting plots are in bounds and do not overlap.
