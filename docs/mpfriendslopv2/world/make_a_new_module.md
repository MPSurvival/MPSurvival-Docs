# Make a new module

A module is one room of a procedural level: a small level made with the kit, and one Data Asset. By the end of this page the generator can draw your room in `L_Procedural`. If you have not read [How procedural levels work](how_procedural_levels_work.md), start there.

- The module levels: `Content/MPFriendslop/Maps/Modules/`
- Their Data Assets: `Content/MPFriendslop/Blueprints/DataAssets/Module/Childs/`
- The kit: `Content/MPFriendslop/Meshes/Environments/Modular/`

---

## Pick which one to start from

Duplicate a shipped module of the same type. It already follows every rule on this page.

| Start from | Type | What it brings |
|---|---|---|
| `L_Module_Hall` | `Normal` | An open hall with four pillars |
| `L_Module_Alcoves` | `Normal` | A cross of corridors and four small rooms |
| `L_Module_Ring` | `Normal` | A loop around a small room with windows and a door |
| `L_Module_FigureEight` | `Normal` | Two loops that cross in the middle |
| `L_Module_Corridor` | `Passage` | A long corridor between two side rooms |
| `L_Module_Chicane` | `Passage` | A zigzag with no line of sight between the exits |
| `L_Module_Vault` | `DeadEnd` | A vault behind a double door, opened by a lever on each side, full of loot |
| `L_Module_Offices` | `DeadEnd` | A dead-end corridor and four offices |
| `L_Module_Spiral` | `DeadEnd` | A spiral that ends on a room behind a door |

---

## Step 1, the room

1. Duplicate the module closest to yours in `Maps/Modules/` and name it `L_Module_<Name>`. Never name a level `Module_<something>`: the generator gives that name to each copy it loads.
2. Build the room with its centre on the level origin. It covers X and Y from `-600` to `600`, floor at Z `0`, ceiling at Z `300`: nine floor slabs and nine ceiling slabs.
3. Add your interior walls, pillars and props, as in [How levels are built](how_levels_are_built.md).

A module holds no `NavMeshBoundsVolume`, no fog, no post-process volume and no `PlayerStart`. `L_Procedural` has them once for the whole map.

---

## Step 2, the rules at the edges

The generator builds the outer sides. Your room has to leave room for them.

| Rule | Why |
|---|---|
| Place no wall on the outer sides | The generator puts walls, corners and doorways there |
| Keep everything inside `-587.8` to `587.8` | The outer wall is `24.4` cm thick, centred on the edge |
| On each exit side, keep the middle cell empty: `400` x `400` cm | That is where the doorway and its door go. A ceiling lamp is fine, nothing on the floor |
| An interior wall may meet an outer side only at `-200` or `200` along that side | That spot is always solid wall. Anywhere else it could land in a doorway |
| Keep walkable, reachable floor within `50` cm of the centre of the room, `(0, 0)`. A thin pillar on the exact centre is fine | The run only starts once the nav mesh reaches it, and the Warden can spawn there |

Keep the centre of every module walkable: the generator only starts the run once the navmesh reaches the centre of each placed module. A module whose centre is blocked or has no floor keeps the run from starting.

The exits depend on the type: `Normal` on all four sides, `Passage` on the east and west sides, `DeadEnd` on the east side only. Build it that way round: the generator turns the room itself.

---

## Step 3, the lamps

The project has no bounce light, so a room without a lamp is black. Every closed room needs its own lamp.

1. Put a `BP_CeilingLight_02` bar in the middle cell of each exit side, along the axis of the exit. Keep it steady: it tells the crew where the way out is.
2. Use `BP_CeilingLight_01` in small rooms, `BP_CeilingLight_03` over a big room such as the vault, and a floor neon to show the floor of a dead-end corner.
3. Make a lamp blink only where steady lamps already light the area. With no bounce light, an area lit by a blinking lamp alone goes black half the time. The shipped rooms blink with `Blinking Interval` at `0.3` and `Blink Randomness` at `1`.

The lamps and their fields are on [Place, switch and make lamps](lamps_and_lights.md).

---

## Step 4, loot and patrol points

1. Place `BP_LootSpawnPoint`s and set `Loot Table` on each, as in [Place loot spawn points and loot tables](../loot/place_loot_spawn_points.md). A dead end is where most of the loot goes: the shipped dead ends have seven points each, the `Normal` and `Passage` modules two to four. Most points use `DA_LootTable_Shelf`. Each dead end also has points on `DA_LootTable_Salvage`, the table with the heavy pieces, the bat and the hammer.
2. In a `Normal` or a `Passage` module, place one `BP_WardenPatrolPoint` on the floor. Put none in a `DeadEnd`.

You never fill the Warden's `Patrol Points` here. The generator collects the points of every module it drew and gives them to the Warden it spawns.

---

## Step 5, the Data Asset

1. In `Blueprints/DataAssets/Module/Childs/`, right click, then **Miscellaneous**, then **Data Asset**.
2. Pick `BP_ModuleDataAsset`. Name it `DA_Module_<Name>`.
3. Fill the three fields.

| Field | What it does | Default |
|---|---|---|
| `Level` | The module level, `L_Module_<Name>` | empty |
| `Module Type` | `Normal`, `Passage` or `DeadEnd`. Decides where the generator can draw it | `Normal` |
| `Weight` | How often it is picked among the modules of its type. `2` is twice as likely as `1` | `1` |

---

## Step 6, add it to the generator

1. Open `L_Procedural` and select `BP_LevelGenerator`.
2. Under `Settings|Generation`, add an entry to `Modules` and pick `DA_Module_<Name>`.

Nothing goes in the packaging settings: the Data Asset brings the module level into a packaged build.

---

## Step 7, test it with two players

1. To see your module in most runs, give it a high `Weight`, for example `100`. Keep the other modules in `Modules`: the generator needs every type to draw a full plan.
2. In the **Play** menu, set **Number of Players** to `2` and **Net Mode** to `Play As Listen Server`.
3. Press Play from `L_Procedural`. Walk your room in the client window: every exit, every loot point, every lamp.

---

## Mistakes that cost time

- **A wall or a prop in a doorway.** Something sits in the middle cell of an exit side. Clear the whole cell.
- **The room is black.** No lamp in it. Every doorway gets a door, so a room cannot borrow light from its neighbour.
- **The module never shows up.** It is not in `Modules`, or the other modules of its type have a much higher `Weight`.
- **The run never starts.** A module has something on its centre, or no floor there, so the nav mesh never reaches it. Clear the centre of every module.
- **A doorway opens onto a void.** A `DA_Module_` has an empty `Level`. The generator skips that cell and the run starts anyway, but the walls and doors around it are still built. Pick the level in the Data Asset.

---

Next: [Tune the generator](tune_the_generator.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
