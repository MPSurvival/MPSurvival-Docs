# Add a placeable structure

Anything the player can buy and put down is one Data Asset pointing at one Blueprint. Eleven ship with the template, from gondolas and the checkout counter to picture frames and a gumball machine.

---

## The recipe

1. Make the actor: a mesh, whatever components it needs, and a **`BP_PlaceableComponent`**.
2. Right click in `Blueprints/DataAssets/Structures/Childs/` → **Data Asset** → `BP_StructureDataAsset`.
3. Name it `DA_Structure_<Thing>` and fill it in.
4. Set `Structure` on the actor's `BP_PlaceableComponent` to that asset.
5. Add it to `AvailableStructures` on `DA_App_Structures`, or it can never be bought.

---

## The fields

| Field | What it does |
|---|---|
| `DisplayName` | The name in the app and on the build tile |
| `Description` | The line under it |
| `PlacedClass` | The actor that gets spawned |
| `Category` | An `E_StructureCategory`, which decides the build menu tab |
| `AllowedZones` | Which `E_StoreZone` values it may be placed in |
| `FootprintCells` | Footprint in cells, as an `IntPoint` |
| `SnapStepCm` | The size of a cell for this structure |
| `RotationStep` | The rotation increment |
| `CanPlaceOnFloor` / `CanPlaceOnWall` / `CanPlaceOnCeiling` | Which surfaces accept it |
| `Cost` | What it costs to buy |
| `PreviewMesh` | The mesh used for the ghost |
| `Icon` | The thumbnail on the tile and in the app list |

`E_StructureCategory` has six values: `Shelf`, `Counter`, `Checkout`, `Storage`, `Decor`, `AdPanel`. Empty tabs are hidden in the build menu, so a category with nothing in it costs nothing.

---

## Getting the footprint right

Physical size is `FootprintCells × SnapStepCm`, so the two are chosen together. Measure the mesh bounds, then pick the step that divides them cleanly:

| Object | Real size | Footprint | Step |
|---|---|---|---|
| Gondola | 100 × 50 | 2 × 1 | 50 |
| Fridge | 125 × 75 | 5 × 3 | 25 |
| Office desk | 122 × 62 | 5 × 3 | 25 |

**Round up, and prefer a small step.** A footprint smaller than the mesh lets two pieces overlap without being refused; a generous footprint on a coarse step leaves a visible gap against the wall — the desk at 3 × 2 on a step of 50 stood 19 cm off the wall, and at 5 × 3 on a step of 25 it sits 6.5 cm off, which reads as a skirting board.

A wall or ceiling footprint is measured **in the plane of the surface**. A 32 × 42 cm picture frame is a footprint on the wall, not on the floor beneath it.

---

## The preview mesh

`PreviewMesh` is what the ghost draws, usually the same mesh as the actor. When the actor is made of several parts, give the ghost a single closed mesh so the outline reads at a glance.

To change the look of build mode, edit `MI_PlacementGhost_Valid`, `MI_PlacementGhost_Invalid` and `MI_PlacementFootprint`.

---

## The icon

Structure icons and app icons are flat white silhouettes on transparency, drawn so every icon in the set carries roughly the same amount of ink. Two rules that matter when you draw one:

- A furniture silhouette needs **feet**. Without two points touching the ground it reads as a hole in a wall rather than as an object.
- Seams between parts are **gaps in the alpha**, not painted lines. A painted line goes wrong the moment the widget tints the icon.

---

## Making a surface a foundation

If your structure is meant to be stood on, like a desk or a counter, its collision has to **block** the `StructureFoundation` trace channel. Otherwise the trace passes through it and the ghost hunts for the floor behind.

If it is not meant to be stood on, leave the channel ignored, which is the default.

---

## Structures placed by hand

Placing an actor in the editor rather than buying it works fine, but set `Structure` on its `BP_PlaceableComponent`, in `Settings|Placement`. Without it, edit mode does not see the actor, `X` does nothing and the piece cannot be sold. It is the field people miss.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
