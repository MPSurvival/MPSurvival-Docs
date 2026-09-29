# The maps, and how a game starts

MPFriendslopV2 ships three maps, plus the rooms the run map is built from. This page says what each one is for, what happens between pressing Play and the end of a run, and where to point the menu at a run map of your own.

They all live in `Content/MPFriendslop/Maps/`.

| Map | What it is |
|---|---|
| `L_MainMenu` | The front end. A menu room with a camera, and the menu widgets on top. It is also the lobby: when a player hosts, this same map reloads as a listen server. |
| `L_Procedural` | The run map. A new floor plan every run, built from the rooms in `Maps/Modules/`. This is where the crew loots, extracts and gets hunted. |
| `Modules/L_Module_*` | The nine rooms the generator loads into `L_Procedural`. Never opened as a run map. |
| `L_Showcase` | A showroom of the level kit meshes, sorted by family. Open it in the editor to browse the pieces. It is not part of the game flow. |

---

## The classes that run each map

The project settings (**Project Settings**, then **Maps & Modes**) are already set:

| Setting | Value |
|---|---|
| `Game Default Map` | `L_MainMenu` |
| `Editor Startup Map` | `L_Procedural` |
| `Game Instance Class` | `BP_FriendslopGameInstance` |
| `Default GameMode` | `BP_FriendslopGameMode` |

`L_MainMenu` overrides the game mode with `BP_MainMenuGameMode` in its World Settings. That mode has no pawn and no HUD, and its `Use Seamless Travel` is on so the other players follow the host into the run.

`L_Procedural` has no override. It uses the project default, `BP_FriendslopGameMode`, which holds the run rules. So a new run map needs nothing in its World Settings. Its `Use Seamless Travel` is on too, so the other players stay connected through a `RESTART`.

`L_MainMenu` holds one `BP_MainMenu` actor. It puts its own camera in view, creates the menu and shows the cursor. Move the actor to frame your menu shot.

---

## From Play to the end of a run

1. The game opens on `L_MainMenu`. The main menu shows `HOST RUN`, `SOLO RUN`, `CUSTOMIZE`, `SETTINGS` and `QUIT`.
2. `SOLO RUN` opens the run map straight away, alone. Nobody can join a solo run.
3. `HOST RUN` opens the session page. The host picks the options and clicks `OPEN LOBBY`. `L_MainMenu` reloads as a listen server, and that is the lobby. Other players find it in the browser and join it.
4. The host clicks `START THE RUN`. Every player travels to the run map together.
5. The run ends when the run clock reaches zero, when every standing player has jumped into the extraction, or when the whole crew is down. With two players or more, a crew that is all down first fights it out in the Last Loser Standing arena, and the run ends after it. The recap screen opens.
6. On the recap, the crew leader clicks `RESTART`: the host, or on a dedicated server the player connected the longest. The run map reloads for everyone with a new floor plan, and a new run starts. The other players see `WAITING FOR THE SERVER TO RESTART` instead of the button.
7. `LEAVE THE RUN`, in the pause menu (`P`), sends you back to `L_MainMenu`. When the host leaves, every other player is sent back too, with the message `THE HOST LEFT THE RUN`. On a dedicated server there is no host: a player who leaves goes back alone, and the others keep playing.

A map change reloads everything. Only `BP_FriendslopGameInstance` survives it: the host options, the join code, the settings and the profile (credits and cosmetics).

The lobby, the join code and the kick are covered in [Sessions and the lobby](../ui/sessions_and_the_lobby.md). What happens during a run is in [How a run works](../loot/how_a_run_works.md).

---

## Play your own run map from the menu

1. Build your level. It needs at least one `PlayerStart`. What else a run map needs is in [Build your own level](../world/build_your_own_level.md).
2. Leave its World Settings alone. The project default game mode is already `BP_FriendslopGameMode`.
3. Open `Content/MPFriendslop/Blueprints/Widgets/Menu/WBP_SessionPage`. Set `Run Level` to your map.
4. Open `WBP_MainMenuPage` in the same folder. Set `Level To Play` to your map.
5. Save both.

Steps 3 and 4 are both needed: `START THE RUN` reads the first one, `SOLO RUN` the second. `RESTART` needs nothing, it reloads whatever map is running.

The map fields are soft references to a level, picked from a list. You never type a path.

| Field | Where | Shipped value | Used by |
|---|---|---|---|
| `Run Level` | `WBP_SessionPage`, in `Settings|Session` | `L_Procedural` | `START THE RUN` |
| `Level To Play` | `WBP_MainMenuPage`, in `Settings|Menu` | `L_Procedural` | `SOLO RUN` |
| `Menu Level` | `WBP_PausePage`, in `Settings|Menu` | `L_MainMenu` | `LEAVE THE RUN` |
| `Menu Level` | `BP_FriendslopPlayerState` (in `Blueprints/PlayerCharacter/`), in `Settings|Session` | `L_MainMenu` | A player the server sends back: kicked, wrong code, host left |
| `Menu Widget Class` | `BP_MainMenu` (in `Blueprints/Misc/`), in `Settings|Menu` | `WBP_MenuRoot` | The menu created on the menu map |

If you replace `L_MainMenu` with your own menu map, set it as `Game Default Map`, give it the `BP_MainMenuGameMode` override, place a `BP_MainMenu` in it, and point both `Menu Level` fields at it.

To change the quota or the run length, see [Change the quota, the run length and the recap](../loot/tune_the_run.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
