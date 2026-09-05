# Folders and naming

Everything the template owns is under `/Game/GrandOpening`. Nothing sits at the root of `Content`, which is what makes it safe to drop into a project of your own and easy to remove again.

---

## The tree

```
/Game/GrandOpening/
├── Blueprints/
│   ├── PlayerCharacter/   BP_StoreCharacter · BP_StorePlayerController · BP_StoreGameMode
│   ├── ActorComponents/   flat. Every component and every manager
│   ├── Misc/              BP_StoreGameInstance · BP_StoreGameState · SG_
│   ├── Environments/      furniture placed in the world, one folder per family
│   │   ├── Boxes/         Childs/
│   │   ├── Checkout/      Childs/ · Parts/Drawer/ · Parts/Payment/
│   │   ├── Screens/       Childs/
│   │   ├── Shelves/       Childs/
│   │   ├── Lights/  Openings/  Build/  Tools/  Sky/  Misc/
│   ├── AI/
│   │   └── <Family>/      the pawn and its controller
│   │       └── Behavior/  BT_ · BB_
│   │           └── Tasks/ BTS_ · BTT_ · BTD_
│   ├── Widgets/           Gameplay/ · World/ · Menu/
│   ├── Menu/              BP_MenuGameMode · BP_MenuPlayerController · BP_MenuCamera
│   ├── Structures/        the S_ structs
│   ├── Enumerations/      the E_ enums
│   ├── Interfaces/        BPI_
│   ├── Functions/         BFL_
│   ├── Animations/        ABP_
│   └── DataAssets/
│       └── <Family>/      BP_<Family>DataAsset  +  Childs/ ← the DA_ instances
├── Meshes/       Environments/ · Props/ · Characters/
├── Materials/    Environments/ · PostProcess/ · Props/ · Widgets/ · Characters/ · Surfaces/
├── Textures/     Widgets/ · Props/ · Characters/ · Icons/ · Fonts/
├── Audios/       Effects/ · Footsteps/<Surface>/ · Widgets/ · Attenuation/
├── DataTables/
├── Inputs/       IMC_ at the root, IA_ in Inputs/
├── Maps/
└── Demo/
```

Four things in there are counterintuitive enough to be worth spelling out.

**`Structures/` holds the structs**, not shop furniture. Furniture is in `Environments/`.

**Data Assets are not in a separate `Data/` folder.** The class and its instances live together: `DataAssets/<Family>/BP_XxxDataAsset` with `<Family>/Childs/DA_Xxx` beside it.

**The game mode and the player controller are in `PlayerCharacter/`**, not in a `Core/` folder.

**`Widgets/` is sorted by space, not by domain.** The question is whose information the widget shows. The till display belongs to the till, so it is in `World/`, even though the drawer readout is drawn in screen space. The carry panel shows your keys, so it is in `Gameplay/`, even though it is drawn beside a box.

---

## Prefixes

One prefix, one class.

| Prefix | Class |
|---|---|
| `BP_` | **Every** Blueprint: actor, component, widget, game mode, Data Asset class |
| `DA_` | A Data Asset instance |
| `E_` | Enumeration |
| `S_` | Struct |
| `BT_` / `BB_` | Behavior Tree / Blackboard |
| `BTT_` / `BTS_` / `BTD_` | Task / Service / Decorator |
| `BFL_` | Blueprint Function Library |
| `BPI_` | Interface |
| `SM_` | Static Mesh |
| `SKM_` / `SKEL_` | Skeletal Mesh / Skeleton |
| `AM_` / `AS_` / `ABP_` / `BS_` | Montage / Sequence / Anim Blueprint / Blend Space |
| `M_` / `MI_` / `MF_` | Material / Material Instance / Material Function |
| `T_` | Texture |
| `F_` / `FF_` | Font / Font Face |
| `SC_` | Sound Cue |
| `WAV_` | Sound Wave |
| `SCLS_` / `SMIX_` / `ATT_` | Sound Class / Sound Mix / Attenuation |
| `PM_` | Physical Material |
| `SG_` | Save Game |
| `DT_` / `CT_` | Data Table / Curve Table |
| `L_` | Level |
| `RT_` | Render Target |

`BP_` is deliberately shared by every Blueprint. Widgets are `BP_*Widget`, not `WBP_`.

`BP_XxxDataAsset` is the class and `DA_Xxx` is the instance. Two different things, two prefixes.

Five prefixes are ambiguous in a lot of Unreal projects, and they were settled here before the first asset was made:

| Ambiguity | Settled as |
|---|---|
| `S_` for Skeleton and for Struct | `S_` is a struct. Skeletons take `SKEL_` |
| `SM_` for Sound Mix and Static Mesh | `SM_` is a static mesh. Sound mixes take `SMIX_` |
| `SC_` for Sound Class and Sound Cue | `SC_` is a sound cue. Sound classes take `SCLS_` |
| `AM_` for Animation Modifier and Montage | `AM_` is a montage |
| `DT_` for Damage Type and Data Table | `DT_` is a data table |

---

## Naming

- **The name describes the thing, not the batch that produced it.** `T_Shelf_Metal`, never `T_Batch2_Shelf`. The same goes for folders: no `Refactor/`, no `Phase3/`.
- **Subject first, qualifier second**: `SM_Shelf_Gondola_100`, `MI_Prop_Metal`, `DA_Product_Milk`.
- **Data Assets carry their family**: `DA_Product_*`, `DA_Structure_*`, `DA_Customer_*`, `DA_Employee_*`, `DA_Unlock_*`, `DA_Ad_*`.
- Dimensions in centimetres, as a suffix: `SM_Mod_Wall_400`.
- No spaces, no accents, no `_Copy` or `_Final`.
- **A function is named after what it returns.** Something called `IsX` returns a bool or is not called `IsX`.

---

## Inside a Blueprint

Two conventions that are as visible as a file name.

**A Blueprint component keeps the exact name of its asset.** A `BP_ViewMotionComponent` on the character is called `BP_ViewMotionComponent`, not `ViewMotion`. Renaming it breaks the link between what you see in the components list and what you open in the content browser.

Engine components are the exception, and they take a name that says their role: `ViewRoot`, `ViewCamera`, `BuyerSpot`. "SceneComponent" describes nothing.

**There are two variable categories, and only one of them is editable.**

| Kind of variable | Category | Instance Editable |
|---|---|---|
| A setting meant for you | `Settings\|<Sub>` | yes |
| Internal runtime state | `No Edit` | no |

`Settings` always takes a subcategory: `Settings|Input`, `Settings|Carry`, `Settings|Checkout`, `Settings|Prompt`. That keeps a thirty-field Details panel readable.

`No Edit` holds caches resolved at `BeginPlay`, phases, spring velocities, previous states. The category states the intent and the flag enforces it.

Open any Blueprint in the project and you should be able to tell in a second what you are allowed to change.

---

## Things you will not find

- No comment nodes in any graph.
- No `Make Literal` nodes. Values are typed straight into the pin they feed.
- No colours built out of floats in a graph.
- No tooltips anywhere.
- No unused variables or dead Data Asset fields. If a field has no reader, it was deleted rather than left for you to wonder about.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
