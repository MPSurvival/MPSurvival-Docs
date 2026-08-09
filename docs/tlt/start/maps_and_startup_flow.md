# Maps, game modes and how the game starts

The game boots into an empty level, that level opens the main menu, and the menu opens the level you play in. There is one game mode for each of those three jobs.

This page shows the chain and the two places you cut into it to run your own level.

- The maps: `Content/TheLastTemplate/Maps/`
- The game modes: `Content/TheLastTemplate/Blueprints/Menu/` and `Content/TheLastTemplate/Blueprints/PlayerCharacter/`

---

## The maps that ship

| Map | What it is for |
|---|---|
| `L_BootMap` | Empty. The first level a packaged game loads. Its only job is to open the main menu. |
| `L_MainMenu` | The menu scene: title screen, options, save slots. No player pawn. |
| `L_ShowcaseMap` | The demonstration level. A linear course of rooms, one system per room, with a panel at the entrance that says what you are looking at. |
| `L_MeshesShowcaseMap` | Every mesh of the environment kit laid out on the ground, and nothing else. Useful when you are dressing a level and want to see a piece before you place it. |

---

## How the game starts

1. The packaged game loads `L_BootMap`, because it is the project's `Game Default Map`.
2. `BP_BootGameMode` runs there. On `Begin Play` it calls `Open Level With Loading Screen` and passes its own `Menu Level Name`, which ships as `L_MainMenu`.
3. `L_MainMenu` runs `BP_MenuGameMode`. It spawns no pawn. `BP_MenuPlayerController` creates the title screen widget and sets input to UI only.
4. The menu creates or picks a save slot, then calls `Launch Slot`, which opens a level with the loading screen. Starting a new game passes the level name written on the node. Continuing passes the level name stored in the slot.
5. The gameplay level runs `BP_TheLastTemplateGameMode`, the project's default game mode. It spawns `BP_PlayerCharacter` with `BP_PlayerController`.

!!! warning "Check the boot map before you package"
    `L_BootMap` ships with a `GameMode Override` in its `World Settings` that points at a game mode that is not in the project. Until you fix it, the boot map does not open the menu. Open `L_BootMap`, open `World Settings`, set `GameMode Override` to `BP_BootGameMode`, and save.

---

## The three game modes

| Game mode | Runs on | `Default Pawn Class` and `Player Controller Class` |
|---|---|---|
| `BP_BootGameMode` | `L_BootMap` | no pawn, engine `PlayerController` |
| `BP_MenuGameMode` | `L_MainMenu` | no pawn, `BP_MenuPlayerController` |
| `BP_TheLastTemplateGameMode` | every other level | `BP_PlayerCharacter`, `BP_PlayerController` |

`BP_TheLastTemplateGameMode` is set once, in **Project Settings**, under **Maps & Modes**, as the `Default GameMode`. The other two are set per level, in `World Settings`, as a `GameMode Override`. That is why `L_ShowcaseMap` and any level you add play the game with no setup: no override means the project default.

`BP_BootGameMode` has one field of its own.

| Field | What it does |
|---|---|
| `Menu Level Name` | The level the boot map opens. Ships as `L_MainMenu`. |

`BP_MenuGameMode` and `BP_TheLastTemplateGameMode` do one thing each on `Begin Play`: they call `Show Loading Screen Until Ready`, which holds the loading screen up until the level has finished warming up, so the player does not walk into a scene that is still popping in. Neither carries any other logic, so a game mode of your own can be a child of one of them and lose nothing.

---

## Opening a level with the loading screen

Two functions live in `BFL_LoadingLibrary`, both under the `Loading Library` category, so you can find them by that name in any graph.

| Function | What it does |
|---|---|
| `Open Level With Loading Screen` | Puts the loading screen up, then opens the level you name. Inputs: `Level Name`, `World Context`. |
| `Show Loading Screen Until Ready` | Puts the loading screen up on arrival and takes it away when the level is ready. Input: `World Context`. |

Use `Open Level With Loading Screen` anywhere you would use `Open Level`. The template calls it from the boot game mode, from the menu when a slot is launched, from the pause menu when you quit to the main menu, and from the death screen when you continue from the last save.

`Level Name` is a name, not a path.

The screen itself is `BP_LoadingScreenWidget`. Its `Fade Speed` and `Spinner Speed` are on the widget defaults if you want to change the pacing.

---

## The game instance

`BP_TLTGameInstance` is the one object that survives a level change. It is declared once for the project, in **Project Settings**, under **Maps & Modes**, as the `Game Instance Class`.

It carries the save.

| Field | What it does |
|---|---|
| `Save Slot Prefix` | The start of the save slot name on disk. Ships as `TLT_Save_`. |
| `Current Slot Index` | The slot being played. `-1` when there is none. |
| `Pending Load Slot` | The slot to restore once the next level has loaded. `-1` when there is none. |
| `Segment Start Seconds` | Play time bookkeeping for the slot header. |

What actually goes into a save, and how to make your own actors survive a reload, is on [Saving a run](../ui/saving_a_run.md).

---

## The two player controllers

`BP_PlayerController` is the controller for gameplay, and it is empty. Everything the player does lives on `BP_PlayerCharacter`, which is described in [How the player character works](../player/how_the_character_works.md). The controller is there so you have somewhere to put controller level logic of your own without touching the character.

`BP_MenuPlayerController` is the controller for `L_MainMenu`. On `Begin Play` it creates the title screen widget, adds it to the viewport and sets input to UI only. If you replace the menu screens, this is the only Blueprint that names the first one. See [The screens and where they live](../ui/screens_overview.md).

---

## Start the game on your own level

Your level needs nothing special. No game mode override, no boot actor.

1. Build your level anywhere under `Content/`.
2. Put a `Player Start` where the player should appear.
3. Leave `World Settings` alone. With no `GameMode Override` the level uses `BP_TheLastTemplateGameMode`, so you get `BP_PlayerCharacter` and the full HUD.
4. Open `BP_MainMenuWidget`, find the `Launch Slot` call on the new game path, and set its `In Level Name` pin to the name of your level.
5. Test by playing from `L_BootMap`, not from your level directly, so you go through the whole chain once.

Continuing a game does not read that pin. It opens the level name stored in the save slot, so an old save still returns to the level it was made in.

---

## Editor Startup Map is not Game Default Map

The two are easy to mix up, and both are in **Project Settings**, under **Maps & Modes**.

- `Editor Startup Map` is the level the editor opens when you open the project. Change it to whatever you are working on, it has no effect on the game.
- `Game Default Map` is the level a packaged game loads first. It is `L_BootMap`, and it should stay `L_BootMap`. Point it straight at a gameplay level and you skip the menu, and with the menu you skip the save slot the rest of the game expects to be set.

If you want a build that skips the menu on purpose, keep `L_BootMap` and change `Menu Level Name` on `BP_BootGameMode` to your level instead. The loading screen and the game instance still get their turn.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
