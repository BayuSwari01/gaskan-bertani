# §4 Game rules

## §4.1 Grid and objects

- The farm is 24×24 tiles. Grid `(x, y)` has its origin at the back corner; tile `(x, y)` covers world
  `[x, x+1] × [y, y+1]` on the XZ plane (grid y maps to world z).
- Footprints: plot 1×1, tree 2×2, animal per definition (ayam 1×1, kambing 2×2, sapi 2×2), building
  per definition. An object's `(x, y)` is the minimum corner of its footprint.
- Objects must be in bounds and must not overlap. At most `MAX_OBJECTS = 300` objects.
- Buildings: exactly one of each, created by `newGame`. They can be moved but never removed.

## §4.2 XP, levels, limits and the level-up bonus

- `level` = the highest L with `levels[L-1].xp ≤ xp`, capped at 20. XP keeps accumulating after 20.
- XP sources (everything else gives 0 XP): watering a plot `xpWaterPlot`; watering a tree the tree's
  `xpWater`; fertilizing `xpFertilize`; harvesting a crop or tree its `xpHarvest`; giving a baby a
  bottle `xpBottle`; feeding an adult its `xpFeed`. XP is clamped at `XP_MAX`.
- Level gates: planting a crop and buying a tree or animal need `level ≥ unlockLevel`
  (`LEVEL_LOCKED`, `detail.level`).
- Plot limit: `plotLimit(level) = min(plotBase + plotPerLevel × (level − 1), plotMax)`; plowing fails
  with `LIMIT_REACHED` (`detail.kind = 'plots'`, `detail.limit`) when the player already owns that many.
- Level-up: after every successful command, for each level crossed in order, add that level's
  `rewardCoins` and `rewardWater` (water may exceed `waterRegenCap`, clamped at `waterHardCap`) and
  emit `levelUp` with the items whose `unlockLevel` equals that level.
- Level-up bonus (FarmVille 2), applied once when at least one level was crossed:
  - every plot whose crop has been watered and is growing or withered gets `wateredAt = now − growMs`
    (ripe now with a full wither window); count `ripened` (was growing) and `revived` (was withered);
  - every tree that is watered and growing gets `wateredAt = now − cycleMs` (fruit ready now);
  - emit `levelBonus { ripened, revived }` plus `objectChanged` with reason `ripened` or `revived` for
    each affected object. Seeded (unwatered) crops, animals and the well are not affected.

## §4.3 Crops and plots

- Phase of a plot, derived from `(plot, now)`; with `e = now − wateredAt`, `g = growMs`,
  `w = witherMs`:
  - empty: `crop === null`;
  - seeded (stage 0, needs water): `crop.wateredAt === null`;
  - growing: `e < g`; stage `1 + floor(3e / g)` (1–3); `remainingMs = g − e`; `progress = e / g`;
  - ripe (stage 4): `g ≤ e < g + w`; `remainingMs = g + w − e` (time until it withers);
  - withered (stage 5): `e ≥ g + w`.
- `plow`: free, 0 XP; creates `{ crop: null, fertilized: false }`.
- `plant`: costs `seedCost`; sets `crop = { defId, plantedAt: now, wateredAt: null }`; 0 XP.
- `water` on a plot: costs 1 air; sets `wateredAt = now`; `+xpWaterPlot`.
- `harvest` on a plot: ripe only; adds `yield` (+1 when the plot is fertilized) of the crop's product
  to the goods; `+xpHarvest`; then `crop = null`, `fertilized = false`.
- `clearPlot`: withered only; `crop = null`, `fertilized = false`; free, 0 XP.
- `revive`: withered only; costs `reviveMultiplier × seedCost`; sets `wateredAt = now − growMs`
  (ripe now with a full wither window); `fertilized` is kept; 0 XP.
- `fertilize` on a plot: allowed when the plot is empty, seeded, growing or ripe (not withered) and
  not yet fertilized; costs 1 pupuk; `+xpFertilize`; the flag waits for the next harvest.

## §4.4 Trees

- A tree is watered once per cycle: needs water (`wateredAt === null`) → `water` costs
  `waterPerCycle` air at once (`+xpWater`) → growing for `cycleMs` → ready → `harvest` adds
  `fruitYield` (+1 when fertilized) of its fruit (`+xpHarvest`), `harvests += 1`, `wateredAt = null`,
  `fertilized = false` → needs water again. Trees never wither.
- Visual, with `p = (now − wateredAt) / cycleMs` while growing:
  - first cycle (`harvests === 0`): needs water → stage 0 (sapling); growing `p < 1/3` → stage 0,
    `p < 2/3` → stage 1, otherwise stage 2; ready → stage 3 with fruit `ripe`;
  - later cycles: always stage 3; fruit `none` while needing water or growing with `p < 0.5`,
    `unripe` while growing with `p ≥ 0.5`, `ripe` when ready.
- `fertilize` on a tree: allowed when not yet fertilized (any phase); costs 1 pupuk; `+xpFertilize`.
- `buyTree` creates `{ plantedAt: now, harvests: 0, wateredAt: null, fertilized: false }`.

## §4.5 Animals

- Baby (`adult === false`): hungry when `bottledAt === null` or `now ≥ bottledAt + bottleIntervalMs`,
  otherwise cooldown. `feed` on a hungry baby costs 1 Botol Susu, `bottlesTaken += 1`,
  `bottledAt = now`, `+xpBottle`. When `bottlesTaken === bottlesToAdult` the animal becomes adult
  (`adult = true`, `fedAt = null`, so it is hungry immediately) and an `objectChanged` event with
  reason `grewUp` is emitted.
- Adult: hungry when `fedAt === null` or `now ≥ fedAt + cooldownMs`, otherwise cooldown. `feed` on a
  hungry adult costs `feedPerMeal` pakan, sets `fedAt = now`, adds `yield` of its product to the goods
  immediately and gives `+xpFeed`. Then it rolls pupuk: `mulberry32Step(state.rng)` gives a value in
  `[0, 1)` and the new rng state; if the value is below `fertilizerChance` and
  `fertilizer < fertilizerCap`, `fertilizer += 1` (event `fertilizer`).
- `buyAnimal` creates a baby `{ boughtAt: now, adult: false, bottlesTaken: 0, bottledAt: null,
  fedAt: null }`; at most `maxOwned` animals of each kind (`LIMIT_REACHED`, `detail.kind = 'animals'`).
- Bubbles: hungry baby → `bottle`; hungry adult → `feed`; otherwise none.

## §4.6 Water (air)

- With `cap = waterRegenCap`: if `amount ≥ cap`, then `k = 0` and `current = amount`; otherwise
  `k = max(0, floor((now − updatedAt) / waterRegenMs))` and `current = min(cap, amount + k)`.
- `nextAt = current ≥ cap ? null : updatedAt + (k + 1) × waterRegenMs`.
- Spending `n` needs `current ≥ n` (else `NOT_ENOUGH_WATER`, `detail.need`, `detail.have`). Then
  `amount = current − n` and `updatedAt = current ≥ cap ? now : updatedAt + k × waterRegenMs` (partial
  progress is kept).
- Adding `n` (well, level-up): first settle as above (`amount = current`, `updatedAt` as for a spend of
  0), then `amount = min(waterHardCap, amount + n)`.
- The HUD shows `current/cap` (for example "12/30", or "45/30" above the cap) and a "+1 mm:ss"
  countdown while below the cap.

## §4.7 Sumur (well)

- Full when `collectedAt === null` or `now ≥ collectedAt + wellRefillMs`; otherwise refilling.
- `collectWell` on the sumur: full only (`NOT_READY`, `detail.ms`); adds `wellWater` air (§4.6 add);
  `collectedAt = now`; 0 XP. Bubble `well` when full.

## §4.8 Goods, selling, milling, bottles and aid

- Goods have no storage limit; each count is an integer between 1 and `GOOD_MAX` (absent when 0).
  An addition beyond `GOOD_MAX` fails with `LIMIT_REACHED` (`detail.kind = 'goods'`).
- `sell`: `qty` is an integer ≥ 1 (`INVALID_QTY`); removes the goods and adds `price × qty` koin
  (beyond `COINS_MAX`: `LIMIT_REACHED`, `detail.kind = 'coins'`); 0 XP.
- `mill`: only goods with `millFeed` (`NOT_MILLABLE`); adds `qty × millFeed` pakan, all or nothing
  against `feedCap` (`FEED_FULL` with `detail.max = floor((feedCap − feed) / millFeed)`).
- `buyBottles`: `qty` is an integer from 1 to 99; costs `qty × bottlePrice`; `bottles + qty` must not
  exceed `bottleCap` (`LIMIT_REACHED`, `detail.kind = 'bottles'`).
- Aid ("Bantuan Desa", the soft-lock guard) is available when all of these hold: koin below the
  cheapest seed cost among unlocked crops; no goods; no plot has a crop that is not withered; no tree
  is watered (growing or ready); `aidClaimedAt` is null or at least `aidCooldownMs` ago. `claimAid`
  adds `aidCoins` and sets `aidClaimedAt = now`; otherwise `AID_UNAVAILABLE`.

## §4.9 Moving and removing

- `move` works for every kind, buildings included. The target footprint must be in bounds and free,
  ignoring the object itself. The object keeps its state.
- `remove` deletes the object. Buildings fail with `NOT_REMOVABLE`. Trees and animals refund
  `floor(cost × removeRefundPercent / 100)` koin; plots refund nothing and lose their crop.

## §4.10 Command summary

Every command fails with `NOT_FOUND` for an unknown id and `WRONG_TARGET` for the wrong kind of
object; an unknown command type gives `UNSUPPORTED`. Level-ups are applied after every successful
command (§4.2).

| Command | Preconditions (error) | Effects | XP |
|---|---|---|---|
| plow | in bounds (OUT_OF_BOUNDS); tile free (OCCUPIED); plot limit and object limit (LIMIT_REACHED) | new empty plot | 0 |
| plant | crop is null (PLOT_NOT_EMPTY); unlocked (LEVEL_LOCKED); koin (NOT_ENOUGH_COINS) | −seedCost; crop planted, unwatered | 0 |
| water (plot) | has a crop (PLOT_EMPTY); not withered (WITHERED); unwatered (ALREADY_WATERED); 1 air (NOT_ENOUGH_WATER) | −1 air; wateredAt = now | xpWaterPlot |
| water (tree) | unwatered (ALREADY_WATERED); waterPerCycle air (NOT_ENOUGH_WATER) | −waterPerCycle air; wateredAt = now | xpWater |
| harvest (plot) | has a crop (PLOT_EMPTY); not withered (WITHERED); ripe (NOT_READY, detail.ms) | +yield goods (+1 if fertilized); crop = null; fertilized = false | xpHarvest |
| harvest (tree) | ready (NOT_READY, detail.ms) | +fruitYield goods (+1 if fertilized); harvests + 1; wateredAt = null; fertilized = false | xpHarvest |
| clearPlot | has a crop (PLOT_EMPTY); withered (NOT_WITHERED) | crop = null; fertilized = false | 0 |
| revive | has a crop (PLOT_EMPTY); withered (NOT_WITHERED); koin (NOT_ENOUGH_COINS) | −reviveMultiplier × seedCost; wateredAt = now − growMs | 0 |
| fertilize | plot not withered (WITHERED); not fertilized (ALREADY_FERTILIZED); 1 pupuk (NOT_ENOUGH_FERTILIZER) | −1 pupuk; fertilized = true | xpFertilize |
| buyTree | unlocked; koin; in bounds; free; object limit | −cost; new tree | 0 |
| buyAnimal | unlocked; koin; maxOwned and object limit (LIMIT_REACHED); in bounds; free | −cost; new baby | 0 |
| buyBottles | qty 1–99 integer (INVALID_QTY); koin; bottleCap (LIMIT_REACHED) | −qty × bottlePrice; +qty bottles | 0 |
| feed (baby) | hungry (NOT_HUNGRY, detail.ms); 1 bottle (NOT_ENOUGH_BOTTLES) | −1 bottle; bottlesTaken + 1; grows up at bottlesToAdult | xpBottle |
| feed (adult) | hungry (NOT_HUNGRY, detail.ms); pakan (NOT_ENOUGH_FEED, detail.need) | −feedPerMeal; fedAt = now; +yield goods; pupuk roll | xpFeed |
| collectWell | target is the sumur (WRONG_TARGET); full (NOT_READY, detail.ms) | +wellWater air; collectedAt = now | 0 |
| sell | qty integer ≥ 1 (INVALID_QTY); enough goods (NOT_ENOUGH_GOODS) | −qty goods; +price × qty koin | 0 |
| mill | millable (NOT_MILLABLE); qty (INVALID_QTY); goods (NOT_ENOUGH_GOODS); room (FEED_FULL, detail.max) | −qty goods; +qty × millFeed pakan | 0 |
| move | in bounds (OUT_OF_BOUNDS); free ignoring itself (OCCUPIED) | x, y updated | 0 |
| remove | not a building (NOT_REMOVABLE) | object deleted; refund for trees and animals | 0 |
| claimAid | §4.8 conditions (AID_UNAVAILABLE) | +aidCoins; aidClaimedAt = now | 0 |

## §4.11 Invariants (`checkInvariants`; also enforced by the save validator)

- `coins`, `xp`, `feed`, `fertilizer`, `bottles`, `water.amount` and every goods count are integers
  ≥ 0; goods entries are ≥ 1 and ≤ `GOOD_MAX`; `coins ≤ COINS_MAX`; `xp ≤ XP_MAX`.
- `water.amount ≤ waterHardCap`, `feed ≤ feedCap`, `fertilizer ≤ fertilizerCap`, `bottles ≤ bottleCap`.
- Every object key equals its `id`, matches `OBJECT_ID_RE` and its number is below `nextId`; at most
  `MAX_OBJECTS` objects; every `defId` and goods key is a known catalog id.
- Footprints are in bounds and do not overlap. Exactly one building of each `BuildingId`.
- Trees: `harvests` is an integer ≥ 0.
- Animals: `0 ≤ bottlesTaken ≤ bottlesToAdult`; `adult` is true exactly when
  `bottlesTaken === bottlesToAdult`; a baby has `fedAt === null`.
- Buildings: `collectedAt` is null unless the building is the sumur.
- Every timestamp is a finite integer in `[TIME_MIN = 2020-01-01, TIME_MAX = 2100-01-01]`; `rng` is an
  integer in `[0, 2^32)`.

## §4.12 State diagrams (informative)

These diagrams summarize §4.3–§4.7. Arrows labelled with a command need a dispatch; arrows labelled
with a duration happen by themselves, because the phase is derived from timestamps (§3.7).

**Plot**

```
 (tile) --plow--> EMPTY --plant (seedCost koin)--> SEEDED --water (1 air, +XP)--> GROWING (stage 1-3)
                    ^                                                               |
                    |                                                         growMs passes
                    |                                                               v
                    +<-------------- harvest (+1 good, +1 more if fertilized, +XP) -- RIPE (stage 4)
                    |                                                               |
                    |                                                         witherMs passes
                    |                                                               v
                    +<----------------------- clearPlot (free) ----------------- WITHERED (stage 5)
                                                                                    |
                                     revive (reviveMultiplier x seedCost) ----------+--> RIPE

 Level-up bonus: GROWING -> RIPE and WITHERED -> RIPE, for every watered crop.
 fertilize: allowed in EMPTY, SEEDED, GROWING and RIPE when not yet fertilized; consumed by harvest,
            lost by clearPlot.
```

**Tree**

```
 buyTree --> NEEDS_WATER --water (waterPerCycle air, +XP)--> GROWING --cycleMs passes--> READY
                 ^                                                                        |
                 +-------------- harvest (+fruitYield, +1 if fertilized, +XP) ------------+

 First cycle (harvests = 0): the sapling shows stage 0, then 1 and 2 while GROWING, and stage 3
 with ripe fruit at READY. Later cycles: always stage 3; no fruit, then unripe fruit from 50%,
 then ripe fruit at READY. Level-up bonus: GROWING -> READY. Trees never wither.
```

**Animal**

```
 buyAnimal --> BABY HUNGRY --feed (1 Botol Susu, +XP)--> BABY COOLDOWN --bottleIntervalMs--> BABY HUNGRY
                                    |
                     the bottlesToAdult-th bottle
                                    v
               ADULT HUNGRY --feed (feedPerMeal pakan): +product now, +XP, maybe +1 pupuk--> ADULT COOLDOWN
                     ^                                                                        |
                     +------------------------------ cooldownMs passes ------------------------+
```

**Sumur**

```
 FULL --collectWell (+wellWater air)--> REFILLING --wellRefillMs passes--> FULL
```
