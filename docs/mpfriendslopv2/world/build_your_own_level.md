# Build your own level

A run map is an ordinary Unreal level with a handful of template actors dropped into it. There is no list to register it in and no setting to add to its World Settings. By the end of this page you have a map the crew can loot, extract from and play from the menu.

The start room of `L_Procedural` and every room in `Maps/Modules/` are built exactly this way. Keep one open next to your level and copy what you need from it.

- The run map: `Content/MPFriendslop/Maps/L_Procedural`
- The rooms: `Content/MPFriendslop/Maps/Modules/`
- The kit: `Meshes/Environments/Modular/`
- The gameplay actors: `Blueprints/Environments/` and `Blueprints/AI/Warden/`

---

## Step 1, the rooms

1. Create a new empty level and save it in `Maps/`.
2. Lay the floor slabs, then the walls, then the ceiling slabs, on the `400` cm grid.
3. Leave a doorway module wherever a door will go, and a hole in the floor where the extraction shaft will go.

The grid, the pivots and the corner rules are on [How levels are built](how_levels_are_built.md). Your own meshes work too: nothing in the gameplay actors cares what the walls are made of.

---

## Step 2, the look

1. Add an `ExponentialHeightFog`.
2. Add a `PostProcessVolume` and tick `Infinite Extent (Unbound)`.
3. In that volume, lock the exposure: set `Min EV100` and `Max EV100` to the same value. `L_Procedural` uses `5`.
4. In the same volume, add `MI_InteractableOutline` to `Post Process Materials`.
5. Place lamps in every room. See [Place, switch and make lamps](lamps_and_lights.md).

The project has no global illumination and no baked lighting, so a room with no lamp is black.

---

## Step 3, the players

1. Place `PlayerStart` actors on the floor, spread apart, at least one per player.

The lobby holds four players by default. `L_Procedural` has `4` of them, in the start room. Nothing else is needed: the project default game mode, `BP_FriendslopGameMode`, spawns the crew and runs the run.

---

## Step 4, the gameplay actors

Place them in this order. Each page has the steps and the fields.

| Actor | What happens without it | Page |
|---|---|---|
| `NavMeshBoundsVolume` over every walkable floor | The Warden cannot walk | [Place a Warden and its patrol](../ai/place_a_warden.md) |
| `BP_LootSpawnPoint`, with a loot table each | Nothing to carry and sell | [Place loot spawn points and loot tables](../loot/place_loot_spawn_points.md) |
| `BP_ExtractionTrap` over a shaft, plus a `BP_InteractButton` linked to it | Nobody can sell or leave. The run only ends on the clock or when the whole crew is down | [Place an extraction point](../loot/place_an_extraction_point.md) |
| `BP_ShopTerminal` | No shop during the run | [Add an item to the shop](../items/the_shop.md) |
| `BP_ReviveBay` | A dead player cannot come back before the next run | [Place a revive bay](../death/place_a_revive_bay.md) |
| `BP_SlidingDoor_Single`, `BP_SlidingDoor_Double`, buttons and levers | Open doorways only | [Link a button or a lever to a door](../interaction/doors_buttons_and_levers.md) |
| `BP_Warden` and its `BP_WardenPatrolPoint`s | No enemy | the Warden page above |
| `BP_ItemPickup` | No item lying on the floor at the start | [Add an inventory item](../items/add_an_item.md) |
| `BP_Arena`, away from the playable space | A crew that is all down ends the run at once, with no arena | [The Last Loser Standing arena](../loot/the_arena.md) |

Beyond the `PlayerStart`s, only the extraction trap and some loot are needed for a real run. Every other line is optional.

`BP_DebugKillVolume`, in `Blueprints/Environments/`, kills any player who enters its box. Put one at the bottom of a pit if your level has one.

---

## Step 5, play it from the menu

Point `Run Level` on `WBP_SessionPage` and `Level To Play` on `WBP_MainMenuPage` at your map. **START THE RUN** travels to the first, **SOLO RUN** opens the second, so set both. The steps are on [The maps, and how a game starts](../start/maps_and_game_flow.md).

To give this map its own quota or run length, see [Change the quota, the run length and the recap](../loot/tune_the_run.md).

---

## Step 6, test it with two players

1. Open your level.
2. In the **Play** menu, set **Number of Players** to `2` and **Net Mode** to `Play As Listen Server`.
3. Press Play and walk the level in the client window: pick up loot, sell a piece at the trap, open a door. Judge from the client, never from the host.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
