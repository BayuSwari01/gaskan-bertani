# Glossary

The game speaks Bahasa Indonesia; code and documents speak English. This table connects the two. Code
ids are lowercase ASCII with underscores (§3.5).

## Content

| Indonesian (display name) | English | Code id | Notes |
|---|---|---|---|
| Tomat | tomato | crop `tomat`, good `tomat` | |
| Padi | rice plant | crop `padi` | its good is Gabah |
| Gabah | unhusked rice grain | good `gabah` | the harvest of Padi |
| Cabai | chili pepper | crop `cabai`, good `cabai` | |
| Jagung | corn, maize | crop `jagung`, good `jagung` | |
| Singkong | cassava | crop `singkong`, good `singkong` | the best feed crop |
| Bawang Merah | shallot (red onion) | crop `bawang_merah`, good `bawang_merah` | |
| Jeruk | orange, citrus | tree `jeruk`, good `jeruk` | |
| Mangga | mango | tree `mangga`, good `mangga` | |
| Rambutan | rambutan | tree `rambutan`, good `rambutan` | red hairy fruit |
| Kelapa | coconut (palm) | tree `kelapa`, good `kelapa` | the only palm |
| Durian | durian | tree `durian`, good `durian` | the most valuable fruit |
| Ayam | chicken (hen) | animal `ayam` | |
| Anak Ayam | chick | baby name of `ayam` | |
| Telur | egg | good `telur` | |
| Kambing | goat | animal `kambing` | |
| Cempe | goat kid | baby name of `kambing` | |
| Susu Kambing | goat milk | good `susu_kambing` | |
| Sapi | cow | animal `sapi` | |
| Pedet | calf | baby name of `sapi` | |
| Susu Sapi | cow milk | good `susu_sapi` | |
| Rumah | farmhouse | building `rumah` | decorative |
| Gudang | barn, storage | building `gudang` | inventory and selling |
| Penggilingan Pakan | feed mill | building `penggilingan` | turns goods into pakan |
| Sumur | well | building `sumur` | +10 air every 4 hours |

## Game terms

| Indonesian | English | Where it appears |
|---|---|---|
| petak | plot | crops grow on plots |
| lahan | farm land | "di luar lahan" = outside the farm |
| bajak, membajak | plow | the Bajak tool creates plots |
| benih | seed | Seed Picker, Toko tab Benih |
| bibit | sapling (young tree) | trees are bought as bibit |
| tanam | plant | "Mode Tanam" |
| siram | water (verb) | "Perlu disiram" = needs water |
| air | water (resource) | HUD, costs |
| tumbuh | grow | "Tumbuh · siap dalam …" |
| siap panen, matang | ripe, ready to harvest | bubbles, tooltips |
| panen | harvest | |
| layu | withered | withered plots |
| bersihkan | clear | clearPlot, free |
| pulihkan | revive | revive, costs koin |
| pupuk, dipupuk | fertilizer, fertilized | the Pupuk tool, the badge |
| pakan | animal feed | HUD, mill |
| giling | grind, mill | the Giling button |
| Botol Susu | milk bottle | raising baby animals |
| beri susu, minta susu | bottle-feed, asks for milk | baby animals |
| beri pakan | feed | adult animals |
| lapar | hungry | |
| kenyang | full, not hungry | cooldown |
| anak, dewasa | baby, adult | animals |
| koin | coins | currency |
| Toko | shop | toolbar |
| jual | sell | Gudang panel, the Jual tool |
| pindah, pindahkan | move | the Pindah tool, object info |
| hapus | delete | removing a plot |
| pasang | place | placement bar |
| batal | cancel | mode chip, placement bar |
| tutup | close | panels |
| lanjut | continue | level-up modal |
| ya, tidak | yes, no | confirmations |
| Naik Level | level up | level-up modal |
| Terbuka | unlocked | level-up modal, locked items |
| Bantuan Desa | village aid | the soft-lock guard |
| Pengaturan | settings | toolbar |
| Suara | sound | settings |
| Kurangi Animasi | reduce motion | settings |
| Mulai Ulang | restart (new game) | settings |
| Segera hadir | coming soon | placeholder toast for unfinished tools |

## Writing Indonesian UI text

- Address the player as "kamu", friendly and short: "Koin tidak cukup.", "Kamu mendapat 52 koin."
- Sentence case; a full stop at the end of full sentences, none on button labels.
- No English in the UI except "XP" and "Level". No emoji.
- Numbers use `Intl.NumberFormat('id-ID')`: 1250 shows as "1.250".
- Durations: "45d" (detik), "4m 05d" (menit), "1j 05m" (jam), "1h 02j" (hari, from 24 hours).
- Every string lives in `src/ui/strings.id.ts`; display names of content live in the catalog (§5).
