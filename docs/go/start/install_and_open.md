# Install and open the project

Everything the template owns lives in one folder: `Content/GrandOpening`. Nothing sits at the root of `Content`, which is what makes it safe to drop into a project you already have.

---

## What you need

- **Unreal Engine 5.8**. `GrandOpening.uproject` is bound to that version.
- Nothing else. No marketplace plugin, and no paid content.

The only engine plugin the template leans on is **Cable Component**, which draws the coiled cord between a checkout and its barcode scanner. It ships enabled with the engine, so there is nothing to turn on.

---

## The folder you get

| Folder | What is in it |
|---|---|
| `Blueprints/PlayerCharacter/` | The character, the controller, the game mode |
| `Blueprints/ActorComponents/` | Every component and every manager |
| `Blueprints/AI/` | The customer and the employee, with their trees |
| `Blueprints/Environments/` | The furniture placed in the world, one folder per family |
| `Blueprints/DataAssets/` | One folder per family: the class, and its `DA_` instances in `Childs/` |
| `Blueprints/Widgets/` | `Gameplay/`, `World/` and `Menu/` |
| `Blueprints/Menu/` | The main menu map's mode, controller and camera |
| `Blueprints/Enumerations/` `Structures/` `Interfaces/` `Functions/` | The `E_`, `S_`, `BPI_` and `BFL_` assets |
| `Blueprints/Misc/` | Game instance, game state, save games |
| `Meshes/` `Materials/` `Textures/` `Audios/` | The art and the sound |
| `Inputs/` | `IMC_Gameplay` and the `IA_` actions |
| `Maps/` | The three levels |

`Blueprints/Structures/` holds the **structs**, not the store furniture. Furniture lives in `Blueprints/Environments/`, sorted by family: `Boxes/`, `Checkout/`, `Shelves/`, `Screens/`, `Lights/`, `Openings/`, `Misc/`.

A Data Asset class is `BP_XxxDataAsset` and its instances are `DA_Xxx`, in the `Childs/` folder beside it. Widgets are `BP_*Widget`.

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

The surface names matter because footstep sounds are chosen from them.

You also need one custom trace channel, `StructureFoundation`, set to **Ignore** by default. That is the channel the build mode traces against to find a floor, a wall or a ceiling. Without it nothing can be placed. See [How placement works](../build/how_placement_works.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
