# FarmVille 2 research

Background for this project: how FarmVille 2 (Zynga, Facebook, 2012) works, and what Gaskan Bertani
copies, adapts or skips. Gathered in October 2026 from the FarmVille 2 fan wiki, Zynga's help center,
launch-era press and guides (sources at the end). Fan-wiki numbers reflect several patches between
2012 and 2018 and sometimes disagree; uncertain points are marked "(?)".

## 1. The game in one paragraph

FarmVille 2 replaced FarmVille 1's flat 2D farm with real-time 3D models seen from a fixed isometric
three-quarter view. The player inherits an overgrown farm, plows plots, plants and waters crops,
grows fruit trees and raises baby animals into adults. Water acts as the game's energy, feed for
animals comes from crops through a feed mill, and coins come from selling goods at a market stand.
In 2014 the mobile spin-off FarmVille 2: Country Escape simplified the loop (no plowing, no
withering, animals eating raw crops).

## 2. Mechanics: FarmVille 2 versus Gaskan Bertani

| Area | FarmVille 2 | Gaskan Bertani |
|---|---|---|
| Plots | placed with the Plow/Plot tool; plowing is free; the plot cap rises with farm expansions | copied; the cap rises with the level (no expansions) |
| Planting | seed bought per plot with coins; drag over many plots | copied |
| Watering | 1 water per plot (+1 XP); the growth timer starts only after watering | copied |
| Growth | real time, 1 minute to 24 hours; crops sway in the wind | copied (FarmVille 2 durations) |
| Harvest | gives the crop and XP, no coins; 1 per plot, 2 when fertilized | copied |
| Withering | ripe crops wither; window disputed (¼ of the grow time, 18–24 h or about 3 days) | copied with fixed windows of 2–24 h per crop |
| Unwither | a premium item, neighbors, or a level-up | revive for 2× the seed cost; level-ups revive everything |
| Drag | click-and-drag to plant, water or harvest many plots | copied (drag-to-act) |
| Trees | coin purchase; water 2–6 per cycle; harvest; water again; never wither | copied (2–4 water per cycle) |
| Groves, heirlooms | bonuses for 4 identical trees, special trees | skipped |
| Baby animals | need baby bottles from friends or real money | bottles bought with koin |
| Adult animals | hungry → feed N feed → product drops at once (+XP, sometimes fertilizer) → cooldown (5 m chickens, 15 m goats, 3 h cows) | copied |
| Prized animals | after many feedings, rarer premium products | skipped |
| Water | regenerates 1 per 3 minutes to 30 (20 at launch); wells give +10 then refill for 4 hours; level-ups give water | copied, with one well |
| Feed | Feed Mill turns harvested goods into a global feed pool (25, more with silos) | copied with a cap of 50, no silos |
| Fertilizer | by-product of feeding animals and of helping neighbors; +1 yield and +1 XP | copied (animals only) |
| Coins | from selling at the Market Stand and level-ups | copied; the Gudang combines inventory and selling |
| Farm Bucks | premium currency | skipped |
| Levels | XP from watering, fertilizing, harvesting and feeding; rewards of coins and water; unlocks | copied with our own curve to level 20 |
| Level-up bonus | every watered crop and tree finishes at once | copied, and withered crops are revived |
| Storage | inventory without a documented cap | copied (no cap) |
| Crafting, mastery, neighbors, quests, expansions | major FarmVille 2 systems | skipped (out of scope) |
| Starting farm | a tutorial farm with 9 plots: 4 ripe tomatoes, 4 seeded, 1 empty | copied |

## 3. FarmVille 2 numbers used as the basis of §5

Each local crop and tree took over the role of a FarmVille 2 item with a similar place in the
progression, keeping its numbers.

| Gaskan Bertani | FarmVille 2 model | FarmVille 2 data (level, cost, time, sell, XP, feed) |
|---|---|---|
| Tomat | Tomato | 1, 10, 1 m, 12, 1, 1 |
| Padi (Gabah) | Wheat | 2, 10, 4 h, 14, 2, 2 |
| Cabai | Blueberry | 3, 15, 2 m, 17, 1, 2 |
| Jagung | Pumpkin | 4, 15, 8 h, 20, 4, 4 |
| Singkong | Strawberry | 5, 18, 24 h, 24, 13, 10 |
| Bawang Merah | Onion | 6, 22, 12 h, 30, 7, 6 |

| Gaskan Bertani | FarmVille 2 model | FarmVille 2 data (level, cost, water per cycle, cycle, fruit, XP, sell) |
|---|---|---|
| Jeruk | Lemon | 2, 260, 3, 12 h, 6, 4, 17 |
| Mangga | Apple | 3, 320, 2, 4 h, 4, 2, 18 |
| Rambutan | Olive | 5, 480, 3, 12 h (we use 8 h), 6, 5, 24–29 |
| Kelapa | Peach | 11, 620, 3, 2 h, 4, 4, 31–33 |
| Durian | Mango | 20, 1800, 3 (we use 4), 2 h (we use 12 h), 4, 10, 98 |

| Gaskan Bertani | FarmVille 2 model | FarmVille 2 data |
|---|---|---|
| Ayam | White Chicken | level 3 (?), 1500 coins, 2 bottles, 3 feed, 5 m, 2 XP, 1 egg (60) |
| Kambing | Red / Saanen Goat | level 1 / 5, 3000 / 9500 coins, 2 / 5 bottles, 4 feed, 15 m, 2–3 XP, milk (85) |
| Sapi | Jersey Cow | level 15, 35000 coins, 9 bottles, 8 feed, 3 h, 7 XP, milk (85) |

Animal prices, unlock levels and bottle counts were lowered for this clone, because it has only three
animals and no premium currency. Bottle intervals are not documented for FarmVille 2 (only a
15-second lockout between bottles), so they are our own choice.

## 4. Visual style notes

- Real-time 3D (Flash 11 Stage3D) in a fixed isometric three-quarter view; zoom and pan, no camera
  rotation; objects could rotate (skipped here).
- Bright, warm, saturated pastoral look; crops sway; animals idle, graze and wander; the world reacts
  to the cursor.
- A water-drop icon over dry seeded crops, a feed bubble over hungry animals, a product icon over
  ready items; harvested goods pop out and bounce; fertilized plots sparkle; the level-up popup lists
  the unlocks.
- Hover popups show information; timers on hover are confirmed for babies and buildings (crops (?)).
- Floating "+coins / +XP" texts are not described for FarmVille 2 (?), but they were used in
  FarmVille 1 and are standard in the genre, so the clone uses them.
- Growth stage art is not documented (?); FarmVille 1 used about four growth bands, which §7.3 follows.

## 5. Sources

- FarmVille 2 wiki (Fandom): https://farmville2.fandom.com/wiki/Seeds,
  https://farmville2.fandom.com/wiki/Water, https://farmville2.fandom.com/wiki/Feed_Mill,
  https://farmville2.fandom.com/wiki/Fertilizer, https://farmville2.fandom.com/wiki/Baby_Bottle,
  https://farmville2.fandom.com/wiki/Levels/1-50, https://farmville2.fandom.com/wiki/Unwither,
  https://farmville2.fandom.com/wiki/Grove, https://farmville2.fandom.com/wiki/Tutorial,
  https://farmville2.fandom.com/wiki/White_Chicken, https://farmville2.fandom.com/wiki/Red_Goat,
  https://farmville2.fandom.com/wiki/Jersey_Cow
- Zynga Help Center: https://zyngasupport.helpshift.com/hc/en/10-farmville-2/faq/116-how-do-i-harvest-crops/,
  https://zyngasupport.helpshift.com/hc/en/10-farmville-2/faq/117-how-do-i-water-crops/,
  https://zyngasupport.helpshift.com/hc/en/10-farmville-2/faq/681-how-do-i-raise-my-animals/?p=all
- Press and guides: https://gamesbeat.com/farmville-2-zynga/,
  https://www.gamezebo.com/walkthroughs/farmville-2-walkthrough/,
  https://kotaku.com/goodbye-farmville-2-its-been-fun-but-ive-had-enough-5945953,
  https://kotaku.com/its-farmville-2-without-all-the-annoying-bits-1564197820,
  https://gamelytic.com/farmville-2-crop-and-tree-harvest-list/
- Country Escape: https://farmvillecountryescape.fandom.com/wiki/Animal,
  https://farmvillecountryescape.fandom.com/wiki/Crop
