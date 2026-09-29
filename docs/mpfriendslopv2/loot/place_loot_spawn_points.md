# Place loot spawn points and loot tables

Loot is not placed by hand. You place `BP_LootSpawnPoint` actors, give each one a loot table, and every run the point rolls a chance and picks an item from its table. By the end of this page you have points in your level and a table of your own.

- The spawn point: `Content/MPFriendslop/Blueprints/Environments/Spawners/BP_LootSpawnPoint`
- The tables: `Blueprints/DataAssets/LootTable/Childs/`
- The loot actors they pick from: `Blueprints/Environments/Loot/Childs/`

---

## Place a spawn point

1. Drag `BP_LootSpawnPoint` into your level.
2. Put it where the item should rest: on the floor, or just above a shelf plank. The arrow is hidden in game.
3. In the Details panel, under `Settings|Spawn`, set `Loot Table`.
4. Adjust the other fields if needed.

When the run starts, the point rolls `Spawn Chance`, picks an item from the table, and moves it straight down onto the first surface under it. The item is set down. It never falls, so it never starts the run damaged.

| Field | What it does | Default |
|---|---|---|
| `Loot Table` | The table this point rolls. Empty means the point spawns nothing | empty |
| `Spawn Chance` | Chance from 0 to 1 that this point spawns anything this run | `0.6` |
| `Random Yaw` | Turns the item to a random direction. Untick it to use the point's own rotation | ticked |
| `Settle Trace Height` | How far above the item's centre, in cm, the search for a surface starts | `25` |
| `Ground Trace Distance` | How far down, in cm, it looks for a surface | `200` |

Every field is set per placed point. Changing a default on the class does not change a point where you already edited that field.

In the rooms of `Maps/Modules/`, most points use `DA_LootTable_Shelf`. Each dead end also has points on `DA_LootTable_Salvage`, the table with the heavy pieces, the bat and the hammer.

---

## Make a loot table

1. In `Blueprints/DataAssets/LootTable/Childs/`, right click, then **Miscellaneous**, then **Data Asset**.
2. Pick `BP_LootTableDataAsset`. Name it `DA_LootTable_<Name>`.
3. Open it and add rows to `Entries`.
4. For each row, set `Loot Class` and `Weight`.
5. Set it as the `Loot Table` of the points that should use it.

| Field | What it does |
|---|---|
| `Loot Class` | The actor to spawn. Usually one of the `BP_Loot_*` classes |
| `Weight` | How often this row is picked. `3` is three times as likely as `1`. `0` never spawns |

The easiest start is to duplicate `DA_LootTable_Salvage` and edit the weights. `DA_LootTable_Shelf` is the same list without `BP_Loot_EngineBlock`, `BP_Loot_Generator` and `BP_Loot_VaultPlate`: heavy items are kept off shelves. `BP_Loot_BaseballBat` and `BP_Loot_Hammer` are in no table. They are placed by hand.

`Loot Class` accepts any actor. To be worth money when sold, it needs a `BP_LootValueComponent`: see [Add a loot item](add_a_loot_item.md).

The pick is the function `PickLootClass` on the table. For another rule (no duplicates, a guaranteed item), make a child of `BP_LootTableDataAsset` and override it.

---

Next: [Place an extraction point](place_an_extraction_point.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
