# How procedural levels work

`L_Procedural` builds a new floor plan at the start of every run. It lays rooms, called modules, on a grid, links every pair of neighbours with a doorway, and closes the rest with walls. One actor owns all of it: `BP_LevelGenerator`, in `Content/MPFriendslop/Blueprints/Environments/Generation/`.

It is the map the game plays: `SOLO RUN` and `START THE RUN` both open it.

This page is the mental model. The pages after it are the recipes.

---

## The grid

| Measure | Value |
|---|---|
| One module | `1200` x `1200` cm, so `3` x `3` cells of the kit |
| The grid | `Grid Size` modules, `5` x `5` by default |
| The start | the bottom row, in the middle |
| Modules per run | `Module Count`, `8` by default, the start not counted |
| Dead ends among them | `Dead End Count`, `2` by default |

The start module is part of `L_Procedural` itself: the `PlayerStart`s, the shop terminal, the revive bay and the extraction trap are there. It has one exit, to the north. Nothing else is ever drawn in the bottom row.

From the start, the generator walks at random, one module at a time, north, south, east or west. The walk can step back onto a module it already drew, so every module is always connected to the start. Then it adds `Dead End Count` dead ends on empty cells that touch exactly one module, never two side by side. Every two modules that touch are linked.

---

## What one module is made of

A module is a small level, `L_Module_<Name>`, built with the kit, plus one Data Asset, `DA_Module_<Name>`, that tells the generator what kind of room it is.

| Type | Exits | Drawn where the grid cell has |
|---|---|---|
| `Normal` | all four sides | three or four neighbours, or two at a right angle |
| `Passage` | east and west | two neighbours on opposite sides |
| `DeadEnd` | east only | one neighbour |

The generator turns each module by a quarter turn so its exits face its neighbours. That is why a dead end is built with its exit on the east side: the generator turns it to face the one room it touches. Dead ends are where most of the loot goes.

---

## Who builds what

| Part | Built by |
|---|---|
| Floor, ceiling, interior walls, props, lamps | the module |
| `BP_LootSpawnPoint`s and `BP_WardenPatrolPoint`s | the module |
| The four outer sides of every module: walls, corners, doorways | the generator |
| A door in every doorway (`Door Chance`, `1` by default) | the generator |
| The Warden, in the module farthest from the start | the generator |

A shared side is always wall, doorway, wall, with the doorway in the middle. A side that faces nothing is three walls. So a module never carries its own outer walls: in the editor, a module level opens without them, and they only appear in play.

---

## Who decides the layout

The server draws the plan and sends it to every player as one replicated value. Each machine, the server included, then builds the same walls and loads the same modules from that plan. No player ever asks for anything, so there is nothing a player can fake.

A player who joins during the run receives the plan when they arrive and builds the level on their own machine. The doors, the loot and the Warden are ordinary replicated actors: they arrive as usual. A door already opened shows open for them, at the right place. That holds for any door or lever built on `BP_GrabAxisComponent`, your own included, placed in a module or spawned during play, with nothing to wire.

Only the server builds the nav mesh, spawns the Warden and sets the quota.

---

## What happens when a run starts

1. Every player sees the loading screen, `WBP_LoadingScreen`. The server draws the plan. Every machine builds the outer walls and starts loading the modules.
2. On each machine, `OnModulesVisible` fires once all its modules are visible.
3. On the server, when every module is in and the nav mesh reaches the centre of each one, the Warden spawns.
4. The server also waits for every player to arrive in the map. Then the quota is set and the clock starts.
5. On each machine, the loading screen closes once the clock has started and that machine's modules are visible.

The quota is not a fixed number here. It is `Quota Ratio` times the value of the loot that actually spawned, so a lucky map asks for more. See [Tune the generator](tune_the_generator.md).

`RESTART` reloads `L_Procedural`, the same way it reloads any run map. The generator draws a new plan, so each run gets a new floor plan.

---

## Limits worth knowing

- One floor. There are no stairs between modules.
- Every module has the same size. A bigger room is several rooms joined by openings inside one module.
- Doorways are single doors, `110` cm wide. The Warden, `220` cm tall, fits through them; a taller enemy needs a taller doorway frame and a taller `Agent Height` on the nav mesh.

---

## Where to go next

- [Make a new module](make_a_new_module.md)
- [Tune the generator](tune_the_generator.md)
- [Doors the Warden can open](../ai/doors_the_warden_can_open.md)

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
