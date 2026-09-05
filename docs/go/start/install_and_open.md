# Install and open the project

Everything the template owns lives in one folder: `Content/GrandOpening`. Nothing sits at the root of `Content`, which is what makes it safe to drop into a project you already have.

---

## What you need

- **Unreal Engine 5.8**. `GrandOpening.uproject` is bound to that version.
- Nothing else. No marketplace plugin, and no paid content.

The only engine plugin the template leans on is **Cable Component**, which draws the coiled cord between a checkout and its barcode scanner. It ships enabled with the engine, so there is nothing to turn on.

---

## The folder you get

```
Content/GrandOpening/
├── Blueprints/
│   ├── PlayerCharacter/   the character, the controller, the game mode
│   ├── ActorComponents/   every component and every manager
│   ├── AI/                the customer and the employee, with their trees
│   ├── Environments/      the furniture you place in the world
│   ├── DataAssets/        one folder per family, class and instances together
│   ├── Widgets/           Gameplay/ World/ Menu/
│   ├── Menu/              the main menu map's mode, controller and camera
│   ├── Enumerations/      the E_ enums
│   ├── Structures/        the S_ structs
│   ├── Interfaces/        the BPI_ interfaces
│   ├── Functions/         the BFL_ function libraries
│   └── Misc/              game instance, game state, save games
├── Meshes/
├── Materials/
├── Textures/
├── Audios/
├── Inputs/
├── DataTables/
├── Maps/
└── Demo/
```

`Blueprints/Structures/` holds the **structs**, not the store furniture. Furniture lives in `Blueprints/Environments/`, sorted by family: `Boxes/`, `Checkout/`, `Shelves/`, `Screens/`, `Lights/`, `Openings/`, `Misc/`. The full naming table is in [Folders and naming](../architecture/naming_and_folders.md).

---

## Press Play, what you should see

The project opens on `L_ExampleMap`, the playable store. Press Play and you spawn as `BP_StoreCharacter` on the shop floor, with the HUD in the top right corner. You can walk, sprint, crouch, pick up boxes, fill shelves, use the office computer, serve a customer and close the store for the day.

If you launch the packaged game instead, you land on `L_MainMenu` first. The three maps are described in [The three maps](maps_and_startup.md).

---

## Dropping it into an existing project

Copy `Content/GrandOpening` into your own `Content` folder, then check three things in **Project Settings**:

| Setting | Value |
|---|---|
| Maps & Modes → Game Default Map | `L_MainMenu` |
| Maps & Modes → Default GameMode | `BP_StoreGameMode` |
| Physics → Physical Surfaces | slots 1 to 5 must be `Concrete`, `Ground`, `Metal`, `Water`, `Wood` |

The surface names matter because footstep sounds are chosen from them. See [Surfaces and sound](../look/surfaces_and_sound.md).

You also need one custom trace channel, `StructureFoundation`, set to **Ignore** by default. That is the channel the build mode traces against to find a floor, a wall or a ceiling. Without it nothing can be placed. See [How placement works](../build/how_placement_works.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
