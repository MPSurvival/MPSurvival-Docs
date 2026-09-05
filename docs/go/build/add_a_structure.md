# Add a placeable structure

Anything the player can buy and put down is one Data Asset pointing at one Blueprint. Eleven ship with the template, from gondolas and the checkout counter to picture frames and a gumball machine.

---

## The recipe

1. Make the actor. A mesh, whatever components it needs, and a **`BP_PlaceableComponent`**.
2. Right click in `Blueprints/DataAssets/Structures/Childs/` → **Data Asset** → `BP_StructureDataAsset`.
3. Name it `DA_Structure_<Thing>` and fill it in.
4. Set `Structure` on the actor's `BP_PlaceableComponent` to that asset, so a copy placed by hand can also be picked up.
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

`E_StructureCategory` has six values: `Shelf`, `Counter`, `Checkout`, `Storage`, `Decor`, `AdPanel`. Empty tabs are hidden in the build menu, so adding a category with nothing in it costs nothing.

---

## Getting the footprint right

Physical size is `FootprintCells × SnapStepCm`, so the two fields are chosen together.

Measure the mesh bounds, then pick the step that divides them cleanly:

| Object | Real size | Footprint | Step |
|---|---|---|---|
| Gondola | 100 × 50 | 2 × 1 | 50 |
| Fridge | 125 × 75 | 5 × 3 | 25 |
| Office desk | 122 × 62 | 5 × 3 | 25 |

Round **up** rather than down. A footprint smaller than the mesh lets two pieces of furniture overlap without being refused; a footprint that is too generous just leaves a visible gap against a wall. The desk was 3 × 2 at a step of 50 and sat 19 cm off the wall, which read as broken. At 5 × 3 with a step of 25 it sits 6.5 cm off, which reads as a skirting board.

A wall or ceiling footprint is measured **in the plane of the surface**, not on the floor. A 32 × 42 cm picture frame is a footprint on the wall, not a footprint underneath it.

---

## The preview mesh

`PreviewMesh` is what the ghost draws, and it is usually the same mesh as the actor. Where they differ is when the actor is made of several parts: give the ghost a single closed mesh so the outline reads at a glance.

The ghost material comes in two instances, `MI_PlacementGhost_Valid` and `MI_PlacementGhost_Invalid`, and the footprint decal underneath uses `MI_PlacementFootprint`. All three are material instances, so changing the look of build mode is a few colour swatches.

---

## The icon

Structure icons and app icons share a look: a flat white silhouette on transparency, drawn so the ink coverage is roughly the same for every icon in the set. That is what stops one tile shouting louder than its neighbours.

Two things that came out of drawing the shipped set, and both apply to any icon you add:

- A furniture silhouette without **feet** reads as a hole in a wall rather than as an object. Two points touching the ground is what says "this is a piece of furniture".
- Seams between parts should be gaps in the alpha, not painted lines. A painted line goes wrong the moment the widget tints the icon.

---

## Making a surface a foundation

If your structure is meant to be stood on, like a desk or a counter, its collision has to **block** the `StructureFoundation` trace channel. Otherwise the trace passes straight through and the ghost hunts for the floor behind it.

If it is not meant to be stood on, leave the channel ignored, which is the default.

---

## Structures placed by hand

Placing an actor in the editor rather than buying it works fine, but set `Structure` on its `BP_PlaceableComponent`. Without it, edit mode does not see the actor, `X` does nothing, and the piece cannot be sold.

It is in `Settings|Placement` in the Details panel, and it is the one field people miss.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
