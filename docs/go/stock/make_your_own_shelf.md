# Build your own shelf

A shelf type is not a Data Asset here, it is a **child Blueprint**. That is the one thing the Data Asset could not give you: a shape.

You duplicate `BP_Shelf_Gondola100`, swap the mesh, drag the splines onto your own shelf boards, and you have your furniture.

---

## The recipe

1. Duplicate `BP_Shelf_Gondola100` into `Blueprints/Environments/Shelves/Childs/` and rename it.
2. Open it and set `ShelfMesh` to your own mesh.
3. Delete or add `BP_StorageRow` components until you have one per shelf board.
4. For each row, drag its spline points onto the **front edge** of its board, and set `DepthCm` and `ClearanceCm`.
5. Make a `DA_ShelfType_<Name>` and put the categories it accepts in `AcceptedCategories`.
6. Set `ShelfType` on your Blueprint to it.
7. If you want to be able to buy and place it, make a `DA_Structure_<Name>` and add it to `DA_App_Structures.AvailableStructures`. See [Add a placeable structure](../build/add_a_structure.md).

---

## Drawing the rows

This is the part that takes the time, and it is the part that pays off.

- The spline runs **along the front edge** of the board. Points are in the actor's local space, so you can drag them in the Blueprint viewport.
- Depth goes toward the spline's **Right** vector. Walk the spline from its first point to its last: depth pushes to your right.
- `DepthCm` is how far back the board is usable. Measure the board, do not guess.
- `ClearanceCm` is the gap up to the next board. On the top row it is whatever headroom you want to allow, which is why the shipped gondola uses `200` up there.

A row does not have to be straight. `BP_Shelf_Round` uses three circular splines for a round display, and the products follow the curve, each one rotated to sit square on the tier it is on.

---

## What the child overrides, and what it does not

The whole system lives on `BP_ShelfBase`. A child sets values, it does not add graphs.

| On the child | Why |
|---|---|
| `ShelfMesh` | A variable, so it can differ per child. A component template cannot |
| The `BP_StorageRow` components | One per board, drawn in the viewport |
| `ShelfType` | Which categories it accepts |
| `InitialProduct` and `InitialUnits` | What it starts stocked with, usually set per placed instance instead |

The interaction, the stocking, the take-back, the display rebuild and the customer-facing queries all come from the base and need no work.

---

## Doors, lids and anything that has to be opened

If your shelf has to be opened before it can be used, override `IsAccessible()` on your child and return your own state.

`BP_Shelf_FridgeTwoDoor` is the shipped example. It inherits the interactable from the base, so `E` on the fridge swings both doors on a timeline, and `IsAccessible` returns `DoorsOpen`. Both stocking gestures and both action lines go away together when the doors are shut, because they all read that one function.

Leave `HasRoomFor` and `HasUnitOf` alone. A closed fridge still has room and still has stock, it just cannot be reached.

---

## Shelf types that ship

| Asset | Accepts |
|---|---|
| `DA_ShelfType_Gondola100` | The dry categories |
| `DA_ShelfType_FridgeTwoDoor` | The chilled categories |

A shelf type has exactly one field, `AcceptedCategories`. If you find yourself wanting to put more on it, ask whether it belongs on the product or on the shelf Blueprint instead.

---

## The mesh side

If you are modelling the furniture rather than reusing a kit piece, three things matter:

- **The pivot goes on the floor**, centred in the footprint. Placement snaps the pivot to the grid.
- **Give it a `UCX_` collision hull.** Without one, the automatic hull will usually swallow the shelf openings and customers will refuse to approach it.
- **Keep the material slots named semantically** (`M_Wood`, `M_Metal`, `M_SteelPainted`), because the props master material is assigned by slot name. See [The mesh kit](../look/mesh_kit.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
