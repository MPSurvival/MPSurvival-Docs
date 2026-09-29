# Tune the generator

Every setting of a procedural level is a field on the generator placed in `L_Procedural`, plus one field on the game mode for the quota. None of it needs a graph. If you have not read [How procedural levels work](how_procedural_levels_work.md), start there.

- The generator: `Content/MPFriendslop/Blueprints/Environments/Generation/BP_LevelGenerator`
- The map: `Content/MPFriendslop/Maps/L_Procedural`

---

## The generator fields

1. Open `L_Procedural` and select `BP_LevelGenerator`.
2. Change the fields under `Settings|Generation`.

| Field | What it does | Shipped in `L_Procedural` |
|---|---|---|
| `Modules` | The `DA_Module_` Data Assets the generator can draw | the nine shipped modules |
| `Module Count` | How many modules a run has, the start not counted | `8` |
| `Dead End Count` | How many of those are dead ends, added at the tips of the level once the rest is laid out | `2` |
| `Grid Size` | How many modules wide and deep the grid is. The start takes the middle of the bottom row, and the rest of that row stays empty | `5` x `5` |
| `Seed` | `0` draws a new plan every run. Any other value draws the same plan every time | `0` |
| `Door Chance` | Chance from 0 to 1 that a doorway gets a door. Below `1`, some doorways only get their frame | `1` |
| `Module Size` | The side of one module, in cm | `1200` |
| `Wall Segment Length` | The length of one outer wall piece, in cm | `400` |
| `Door Class` | The door spawned in a doorway | `BP_SlidingDoor_Single` |
| `Enemy Class` | The enemy spawned in the module farthest from the start. Empty means no enemy | `BP_Warden` |
| `Patrol Point Class` | The class of the points collected for its patrol | `BP_WardenPatrolPoint` |

`Door Class`, `Enemy Class` and `Patrol Point Class` are empty on the Blueprint itself and set on the generator placed in `L_Procedural`. A generator you place in your own map needs them too.

`Module Count` cannot go above the grid: with a `5` x `5` grid, the most is `20`. Keep it at `2` or more, or the Warden gets no patrol point: a single module is always a dead end.

A dead end only goes on an empty cell that touches exactly one module, and two dead ends never touch. On a small grid there is not always room for all of them: the run then has fewer modules, with no error. For more dead ends, raise `Grid Size` too.

`Seed` only fixes the plan. The loot still changes every run, because each `BP_LootSpawnPoint` rolls on its own.

The outer wall meshes are on the generator's four components, `Walls`, `Doorways`, `Corners` and `DoorFrames`. Select one in the **Components** panel and swap its `Static Mesh` to use your own pieces. Keep `Module Size` and `Wall Segment Length` in step with them.

---

## The quota

On a procedural map, the quota follows the loot that actually spawned.

1. Open `BP_FriendslopGameMode`.
2. In `Settings|Extraction`, set `Quota Ratio`.

| Field | What it does | Default |
|---|---|---|
| `Quota Ratio` | The quota is this value times the value of all the loot spawned in the level | `0.6` |

`Quota Ratio` only applies on a map that has a generator. A map with no generator, such as a hand built map of your own, keeps its fixed `Run Quota`. The other run numbers are on [Change the quota, the run length and the recap](../loot/tune_the_run.md).

---

## A bigger grid

The Warden only walks where the nav mesh is. `L_Procedural` has one `NavMeshBoundsVolume` sized for a `5` x `5` grid. When you raise `Grid Size`, stretch that volume over the new modules. Keep its bottom just under the floor: deeper, it would also cover the shaft under the extraction trap.

---

## The dispatchers

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnModulesVisible` | All the modules are visible on this machine | every player, not a dedicated server |
| `OnGenerationFinished` | Every module is in, the nav mesh reaches the centre of each one, and the enemy has spawned | the server only |

Bind `OnModulesVisible` for anything a player sees, such as a sound or a HUD message. Bind `OnGenerationFinished` for anything that needs the nav mesh.

The loading screen binds neither. `WBP_LoadingScreen` asks every level builder `IsLevelVisible`, a few times a second, until all of them answer true and the clock has started. To use your own screen, set `Loading Widget Class` on `BP_FriendslopPlayerController`.

---

## Your own generator

The game mode does not know `BP_LevelGenerator`. It waits for any actor that implements `BPI_LevelBuilder`.

1. Add `BPI_LevelBuilder` to your actor, and return true from `IsLevelReady` once your level is complete.
2. When it is, get the game mode and send it `NotifyLevelReady`, from `BPI_RunControl`, on the server.
3. On every machine with a screen, return true from `IsLevelVisible` once the level can be seen there. The loading screen waits for it.

Until then the run does not start: no quota, no clock. A map with no level builder starts the run once every player has arrived.

---

Next: [Doors the Warden can open](../ai/doors_the_warden_can_open.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
