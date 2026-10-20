# Game design

This document explains what Gaskan Bertani is and why it works the way it does. The rules themselves
are in `spec/` (§4 and §5); the numbers below are examples taken from §5.

## 1. Pitch

"Gaskan Bertani" ("let's go farming") is a cozy farming game in the spirit of FarmVille 2, set on a
small Indonesian village farm. The player plows plots, plants tomat, padi, cabai, jagung, singkong and
bawang merah, grows jeruk, mangga, rambutan, kelapa and durian trees, and raises ayam, kambing and sapi
from babies into adults that produce telur and susu. Everything happens on one isometric 3D farm that
keeps living while the browser is closed.

The player fantasy: "my own little kampung farm that is always a little better than yesterday". Short
satisfying click sessions now, and a reason to come back later.

## 2. Design pillars

1. **Satisfying clicks.** Every action answers immediately and visibly: the model changes, goods pop
   out, numbers float up. Sweeping the mouse across a field to water or harvest many plots in one
   gesture (drag-to-act) is the signature FarmVille 2 feeling.
2. **Come back later.** Real FarmVille 2 durations (from 1 minute up to 24 hours) and withering
   give the farm a rhythm: plant short crops while playing and long crops before leaving. Air (water)
   refills only up to 30, so the farm rewards regular, short visits instead of marathon sessions.
3. **An interlocking economy.** Crops feed animals, animals give fertilizer to crops, trees turn water
   into koin. Every resource has a producer and a consumer (section 4).
4. **Readable and cozy.** Bright low-poly models, a fixed isometric camera, bubbles that say what an
   object needs, and Indonesian text that is short and friendly.
5. **Fair single player.** No premium currency, no energy bar except water, no neighbors needed.
   Things FarmVille 2 sold for real money (milk bottles, unwither) cost koin here, and "Bantuan Desa"
   makes sure the player can never get stuck.

## 3. The economy at a glance

```
                  +--------------------------- koin <---------------------------+
                  |                                                             |
                  v                                                             |
   Toko: benih, bibit pohon, anak hewan, Botol Susu                    Gudang: jual
                  |                                                             ^
                  v                                                             |
   PETAK: tanam -> siram (1 air) -> tumbuh -> panen -----------> goods ---------+
   POHON: siram (N air) -> berbuah -> panen ---------------------> buah --------+
                                                  |                             |
                       tanaman & buah  -> Penggilingan Pakan -> pakan           |
                                                                    |           |
   HEWAN: anak + Botol Susu xN -> dewasa -> beri pakan -> telur / susu ---------+
                                                    \
                                                     +-> kadang pupuk -> petak / pohon (+1 hasil)

   air:  +1 every 3 minutes up to 30, Sumur +10 every 4 hours, level-up rewards
   XP:   watering, fertilizing, harvesting, bottles, feeding -> levels -> unlocks + koin + air
```

## 4. Resources and their roles

| Resource | Gained from | Spent on | Limit | Design role |
|---|---|---|---|---|
| koin | selling goods in the Gudang, level-up rewards, refunds, Bantuan Desa | seeds, trees, baby animals, Botol Susu, reviving withered crops | none in practice | the score of the farm and the only way to grow it |
| XP / level | watering, fertilizing, harvesting, bottles, feeding | nothing | levels 1–20 | unlocks content, raises the plot limit, gives rewards and the level-up bonus |
| air | +1 per 3 minutes up to 30, Sumur +10 per 4 hours, level-ups | watering crops (1) and trees (2–4 per cycle) | regenerates to 30, can go up to 999 | FarmVille 2's "energy": paces active play and rewards regular visits |
| pakan | Penggilingan Pakan (crops and fruit) | feeding adult animals | 50 | turns plant goods into much more valuable animal goods |
| pupuk | feeding adult animals (chance) | fertilizing plots and trees | 99 | links animals back to crops; +1 good per harvest |
| Botol Susu | bought in the Toko (20 koin) | raising babies | 999 | the "raising" part of animals; a small koin sink |
| goods | harvesting, feeding animals | selling, milling | none | stored in the Gudang until sold or milled |

## 5. The three loops, worked out

All numbers come from §5. If §5 changes, recompute the examples.

### 5.1 Crops

**Tomat, the first crop.** At 10:00:00 the player plants a Tomat (−10 koin) and waters it at 10:00:05
(−1 air, +1 XP). The timer starts at watering, so it is ripe at 10:01:05 and withers at 12:01:05 if it
is not harvested. Harvesting gives 1 Tomat and +1 XP; selling it gives 12 koin. Net result: +2 koin,
+2 XP, −1 air. Tomat is the "active play" crop: fast, but it burns water.

**Padi, a long crop.** Planted and watered at 21:00 (−10 koin, −1 air, +1 XP), ripe at 01:00, withers
at 09:00. Coming back in the morning before 09:00 gives 1 Gabah (+2 XP, sells for 14 koin). One
watering earns +4 koin and 3 XP: much better per air than Tomat.

**Per air, the crops compare like this** (one watering each, no fertilizer):

| Crop | Unlock | Ripe after | Withers after ripe | Profit per air | XP per air | Pakan if milled |
|---|---|---|---|---|---|---|
| Tomat | 1 | 1 m | 2 j | +2 koin | 2 | 1 |
| Padi | 2 | 4 j | 8 j | +4 koin | 3 | 2 |
| Cabai | 3 | 2 m | 2 j | +2 koin | 2 | 2 |
| Jagung | 4 | 8 j | 12 j | +5 koin | 5 | 4 |
| Singkong | 5 | 24 j | 24 j | +6 koin | 14 | 10 |
| Bawang Merah | 6 | 12 j | 18 j | +8 koin | 8 | 6 |

The intent, as in FarmVille 2: crops are mainly for XP and for feed, not for koin. Long crops are
water-efficient and XP-rich; short crops let the player keep clicking during a session.

**Fertilizer** adds one good to the next harvest of that plot (Tomat gives 2 instead of 1) and +1 XP
when it is applied. Because it comes from animals, a player with animals gets better crops.

**Withering** gives long crops their stakes: a ripe crop waits for a while (2 to 24 hours, see §5.3),
then withers and yields nothing. A withered plot can be cleared for free or revived for twice the
seed cost (a single-player replacement for FarmVille 2's paid "Unwither"). Reaching a new level also
revives every withered crop, which turns a level-up into a small rescue.

### 5.2 Trees

**Mangga.** Bought for 320 koin at level 3 and placed on a 2×2 spot. Watering costs 2 air (+1 XP).
Four hours later the sapling has grown into a full tree with 4 ripe mangoes; harvesting gives 4 Mangga
(+2 XP, 72 koin if sold) and the tree needs water again. It pays back after about 5 cycles.

| Tree | Cost | Air per cycle | Cycle | One harvest is worth | Value per air | Cycles to pay back |
|---|---|---|---|---|---|---|
| Jeruk | 260 | 3 | 12 j | 6 × 17 = 102 koin | 34 | 3 |
| Mangga | 320 | 2 | 4 j | 4 × 18 = 72 koin | 36 | 5 |
| Rambutan | 480 | 3 | 8 j | 6 × 24 = 144 koin | 48 | 4 |
| Kelapa | 620 | 3 | 2 j | 4 × 32 = 128 koin | 43 | 5 |
| Durian | 1800 | 4 | 12 j | 4 × 98 = 392 koin | 98 | 5 |

The intent: trees turn water into koin far better than crops, but they cost koin up front and need
space. They never wither, so they are the relaxed, reliable income of the farm.

### 5.3 Animals

**Ayam.** Unlocked at level 3 and bought for 600 koin as an Anak Ayam (a yellow chick). The chick
needs 2 Botol Susu (20 koin each) at least 5 minutes apart (+1 XP each). After the second bottle it is
an adult hen and hungry right away. Feeding costs 3 pakan and immediately gives 1 Telur (+2 XP, sells
for 60 koin) with a 20% chance of 1 pupuk. Five minutes later it is hungry again.

Where the pakan comes from: 1 Singkong milled gives 10 pakan, enough for three feedings. Three Tomat
(worth 36 koin) milled give only 3 pakan, which is still enough for one Telur worth 60 koin. Animals
always turn plant goods into more value; that is why the Penggilingan Pakan exists.

| Animal | Cost incl. bottles | Grows up after | One feeding | Cooldown | Pupuk chance |
|---|---|---|---|---|---|
| Ayam | 600 + 2 × 20 | 2 bottles, 5 m apart | 3 pakan → 1 Telur (60) | 5 m | 20% |
| Kambing | 2000 + 3 × 20 | 3 bottles, 15 m apart | 4 pakan → 1 Susu Kambing (85) | 15 m | 30% |
| Sapi | 6000 + 6 × 20 | 6 bottles, 1 j apart | 8 pakan → 2 Susu Sapi (170) | 3 j | 50% |

The intent: animals are the long-term goal and the strongest engine of the farm. Babies make the
purchase feel like raising a pet rather than buying a machine.

### 5.4 Water and the well

Air works like FarmVille 2's energy. It refills by itself up to 30 (one every 3 minutes), so a player
who is away for 90 minutes finds a full tank, while a player who is away for 10 hours finds the same
full tank. The Sumur gives +10 air every 4 hours when tapped, and level-ups give 10–30 air that may go
above 30. Short crops spend water quickly; long crops and trees spend it efficiently. This is the
main lever that makes the player think instead of clicking blindly.

## 6. Progression

| Level | XP | Unlocks | Reward |
|---|---|---|---|
| 1 | 0 | Tomat | – |
| 2 | 20 | Padi, Jeruk | 100 koin, 30 air |
| 3 | 80 | Cabai, Mangga, Ayam | 200 koin, 30 air |
| 4 | 180 | Jagung | 300 koin, 30 air |
| 5 | 330 | Singkong, Rambutan, Kambing | 650 koin, 20 air |
| 6 | 580 | Bawang Merah | 750 koin, 10 air |
| 7 | 980 | Kelapa | 850 koin, 10 air |
| 8 | 1580 | Sapi | 950 koin, 10 air |
| 10 | 3530 | Durian | 1150 koin, 10 air |
| 11–20 | up to 35130 | – | 1250–2400 koin, 10–15 air |

Each level also raises the plot limit by 4 (12 at level 1, up to 80), and the level-up bonus ripens
every watered crop and tree on the spot.

Pacing intent (a guide for balancing, not a test): the first session of 15–20 minutes reaches level
3 with 12–16 plots and a first long crop planted before leaving; the first day reaches level 4–5 and
the first hen; days 2–4 bring the kambing, singkong for feed and regular pupuk; the sapi and durian
arrive in the second week.

## 7. The first session (the FarmVille 2 tutorial farm)

1. The game opens on a small farm: Rumah, Gudang, Penggilingan Pakan and Sumur along the back, and in
   the middle 9 plots: 4 ripe tomatoes with "ready" bubbles, 4 freshly seeded plots with water-drop
   bubbles, and 1 empty plot. 150 koin, 30 air, 2 pupuk.
2. Hint: "Panen 4 tomat yang sudah matang. Klik tomatnya!" The player clicks (or sweeps) across the
   ripe tomatoes. Each one squashes, a Tomat flies to the Gudang button, "+1 Tomat" and "+1 XP" float
   up.
3. Hint: "Siram 4 tomat yang baru ditanam." A sweep waters them; the soil turns dark and sprouts
   appear. A minute later they are ripe.
4. Hint: "Tanam benih di petak yang kosong." Clicking the empty plot opens the Seed Picker; choosing
   Tomat plants it and keeps the player in "Mode Tanam: Tomat".
5. Hint: "Jual tomat di Gudang untuk mendapat koin." The Gudang panel sells them.
6. Hint: "Pilih Bajak, lalu klik rumput untuk membuat petak baru." The player plows plots 10 to 12.
7. Around 20 XP the "Naik Level 2!" modal appears: +100 koin, +30 air, Padi and Jeruk unlocked, and
   every watered tomato is ripe at once. The player keeps playing, buys a Jeruk tree, and before
   leaving plants Padi, which will be ripe by the next visit.

## 8. Signature moments to get right

- Sweeping across a field: every plot reacts in turn, like FarmVille 2's drag harvest.
- Goods popping out and flying to the Gudang button.
- The level-up modal followed by the whole field ripening at once.
- Bubbles that tell what each object needs (water drop, product icon, bottle, pakan sack, wilted leaf).
- Hover tooltips with live countdowns ("Tumbuh · siap dalam 3m 05d").
- Gold sparkles on fertilized plots and trees.
- Hens wandering and pecking, goats and cows ambling around inside their own footprint (there are no
  pens or fences around animals).

## 9. Deliberately left out

Farm expansion, crafting in a kitchen, crop mastery, groves, prized animals, neighbors, quests, premium
currency, object rotation and storing objects in an inventory (§1). They are FarmVille 2 features, but
the goal of this project is the core loop of plots, trees and animals, done well.
