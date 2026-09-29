# How a run works

A run is one trip into the map. Loot appears, the crew carries it to the extraction trap and sells it. Once the team has sold enough to meet the quota, everyone jumps into the shaft to get out.

The rules live on `BP_FriendslopGameMode`. The shared state lives on `BP_FriendslopGameState`. The value of each item lives on its own `BP_LootValueComponent`.

- The GameMode: `Content/MPFriendslop/Blueprints/PlayerCharacter/`
- The GameState: `Content/MPFriendslop/Blueprints/Misc/`
- The loot actors: `Content/MPFriendslop/Blueprints/Environments/Loot/`

This page is the mental model. The pages after it are the recipes.

---

## The loop

1. **Loot spawns.** When the map starts, each `BP_LootSpawnPoint` rolls its `Spawn Chance`, picks an item from its loot table, and places it on the floor or shelf under it.
2. **You carry it.** Loot is a physics object that you pick up with the grab system.
3. **It gets damaged.** Every hard hit lowers the item's condition, and with it the item's price. You see a red flash and a `-$N` number, and enemies hear the hit.
4. **You sell it at the trap.** Put the loot on `BP_ExtractionTrap` and hold the button next to it. The floor opens, the loot falls into the chute and is sold into the team bank.
5. **The quota is met.** The trap opens for good, the sign reads `JUMP IN!` and the HUD tells everyone to go to the extraction point and jump.
6. **You jump.** Each player who falls down the shaft is out of the run and becomes a spectator. The items in that player's inventory, and the one in their hands, are destroyed: only what was sold counts.
7. **The recap.** When the run ends, every player sees the recap screen: what was sold, what was left behind, the total against the quota, and each player's share. It reads `YOU MADE IT OUT` only when the quota was met and at least one player jumped out, and `QUOTA NOT MET` otherwise. After an arena, it also lists each player's kills and the winner.

There is one run per match. On the recap, only the crew leader has the `RESTART` button: it reloads the run map for everyone with fresh loot, and on `L_Procedural` with a new floor plan. On a listen server, the crew leader is the host. On a dedicated server, it is the player connected the longest; when that player leaves, the next one takes over, even with the recap already open. The other players see a line telling them to wait for the server to restart.

---

## What an item is worth

The price is set in the item's `DA_Loot_*` Data Asset, and lowered by the item's `Condition`, which goes from 1 (perfect) to 0.

**Value = `Base Value` x `Condition`, rounded.**

| Field | What it does |
|---|---|
| `Base Value` | The price in perfect condition |
| `Fragility` | The share of the value that one very hard hit removes, from 0 to 1. At `0` the item never loses value |
| `Impact Speed Threshold` | Below this impact speed, in cm/s, a hit costs nothing |
| `Reference Impact Speed` | The impact speed that costs exactly `Fragility`. Between the two speeds, the loss grows in a straight line |

An item can never lose more than it is worth. A fragile item can also shatter and be lost for good.

Loot lying on the trap takes no damage.

---

## How a run ends

The GameMode ends the run on the first of these three:

| End | When |
|---|---|
| The clock | `Run Duration Seconds` runs out. It ships at `300` (5 minutes) |
| Everyone is out | Every player still standing has jumped. At least one player must be out, and downed players do not hold the crew back |
| Everyone is down | The whole crew is down at the same time, with no arena to go to |

With two players or more, a crew that is all down first falls into the Last Loser Standing arena, and the run ends when the arena is over. See [The Last Loser Standing arena](the_arena.md).

When the run ends, the GameMode adds up the value of every item still in the map. That is the `LEFT BEHIND` line on the recap.

The quota ships at `2000`. Both numbers are on the GameMode.

---

## After the run: credits

At the end of the run, the team's extracted value is split evenly between the players. Each player's share becomes **credits** in their own profile, which they spend on cosmetics in the main menu. What a player earned in the arena is added to their share. See [How cosmetics work](../crew/how_cosmetics_work.md).

The shop terminal spends from the same team bank during the run, so buying supplies slows down the quota. See [Add an item to the shop](../items/the_shop.md).

---

## Who decides

| What | Where it lives | Who decides |
|---|---|---|
| Spawning loot | `BP_LootSpawnPoint` | The server. Spawned loot reaches every player |
| An item's condition | `BP_LootValueComponent` | The machine that moves the item detects the hit, and the server works out the loss. A player who joins late sees damaged items at their real value |
| Selling | `BP_ExtractionTrap` | The server |
| The bank, the quota and the manifest | `BP_FriendslopGameState` | The server writes them, every player reads them |
| When the run ends, restarting | `BP_FriendslopGameMode` | The server. It refuses a restart from anyone but the crew leader, `Crew Leader` on `BP_FriendslopGameState` |
| The run clock | `BP_FriendslopGameState` | One end time is shared, and each machine counts down from it |
| Credits | Each player's profile | Given by the server, saved on each player's own machine |

The server picks the crew leader in `UpdateCrewLeader`, on `BP_FriendslopGameMode`, when the run ends and when a player joins or leaves. To pick the leader another way, a vote for example, rewrite that function: the button and the restart request read the result and need no change.

The widgets never cast to these classes. They read the GameState through two interfaces, `BPI_ExtractionBank` and `BPI_RunState`, so your own GameState only has to implement them. See [Use your own game framework classes](../ui/use_your_own_framework.md).

To react to the run in your own HUD or sounds, bind these on the GameState:

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnExtractionTotalsChanged` | The extracted value changed. Gives `Extracted Value` and `Quota` | every player |
| `OnRunPhaseChanged` | The run ended. Gives `Run Ended` | every player |

The rules behind this are on [How multiplayer works](../start/how_multiplayer_works.md).

---

## Where to go next

- [Add a loot item](add_a_loot_item.md)
- [Make a loot item breakable](make_loot_breakable.md)
- [Place loot spawn points and loot tables](place_loot_spawn_points.md)
- [Place an extraction point](place_an_extraction_point.md)
- [Change the quota, the run length and the recap](tune_the_run.md)
- [The Last Loser Standing arena](the_arena.md)

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
