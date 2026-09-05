# Build your own shelf

A shelf is a child Blueprint, because what changes from one shelf to the next is a **shape**. You duplicate the gondola, swap the mesh, drag the splines onto your own boards, and you have your furniture.

---

## The recipe

1. Duplicate `BP_Shelf_Gondola100` into `Blueprints/Environments/Shelves/Childs/` and rename it.
2. Open it and set `ShelfMesh` to your own mesh.
3. Delete or add `BP_StorageRow` components until you have one per shelf board.
4. For each row, drag its spline points onto the **front edge** of its board, and set `DepthCm` and `ClearanceCm`.
5. Create a `DA_ShelfType_<Name>` and list the categories it accepts in `AcceptedCategories`.
6. Set `ShelfType` on your Blueprint to it.
7. To be able to buy and place it in game, create a `DA_Structure_<Name>` and add it to `DA_App_Structures.AvailableStructures`. See [Add a placeable structure](../build/add_a_structure.md).

---

## Drawing the rows

- The spline runs **along the front edge** of the board. Points are in local space, so you drag them in the Blueprint viewport.
- Depth goes toward the spline's **Right** vector: walk the spline from its first point to its last, and depth pushes to your right.
- `DepthCm` is how far back the board is usable. Measure the board.
- `ClearanceCm` is the gap up to the next board. On a top row it is whatever headroom you allow — the shipped gondola uses `200`.

A row does not have to be straight. `BP_Shelf_Round` uses three circular splines, and the products follow the curve.

---

## What you set on the child

| On the child | |
|---|---|
| `ShelfMesh` | Your mesh |
| The `BP_StorageRow` components | One per board, drawn in the viewport |
| `ShelfType` | Which categories it accepts |
| `InitialProduct` and `InitialUnits` | What it starts stocked with, usually set per placed instance instead |

The interaction, the stocking, the take-back and the display all come from `BP_ShelfBase`. There is no graph to write.

---

## Doors and lids

If your shelf has to be opened before it can be used, override `IsAccessible()` on your child and return your own state. Both stocking gestures and both prompt lines disappear while it returns `false`.

`BP_Shelf_FridgeTwoDoor` is the shipped example: `E` swings both doors, and `IsAccessible` returns `DoorsOpen`.

---

## Shelf types that ship

| Asset | Accepts |
|---|---|
| `DA_ShelfType_Gondola100` | The dry categories |
| `DA_ShelfType_FridgeTwoDoor` | The chilled categories |

A shelf type has one field, `AcceptedCategories`.

---

## If you are modelling the furniture

- **Put the pivot on the floor**, centred in the footprint. Placement snaps the pivot to the grid.
- **Give it a `UCX_` collision hull.** The automatic hull swallows the shelf openings, and customers then refuse to approach it.
- **Name the material slots semantically** (`M_Wood`, `M_Metal`, `M_SteelPainted`): the props master material is assigned by slot name. See [The mesh kit](../look/mesh_kit.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
