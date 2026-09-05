# How shelves and storage work

Every place a product can sit — a gondola, a round display, a fridge, the inside of a cardboard box — is built from the same two components: `BP_StorageComponent`, and one or more `BP_StorageRow`.

---

## A row is a spline you draw

`BP_StorageRow` is a spline component. You draw it along the front edge of a shelf board in the viewport and it works out on its own how many units fit, from the size of whatever product is on it. Nothing about the layout is written down anywhere: change the product and the row re-slices itself.

| Field | What it does |
|---|---|
| `DepthCm` | How far back from the curve the row extends |
| `ClearanceCm` | The free height above the row |

**Two things to get right, and they are the only two:**

1. **Draw the spline along the front edge, left to right.** Depth pushes toward the spline's Right vector. A row drawn backwards fills toward the front of the shelf and the products hang in the air.
2. **`ClearanceCm` really refuses things.** A 34 cm cereal box will not go on a row with 35 cm of clearance once the shelf board is accounted for, but it goes on the top row where there is nothing above it.

A row does not have to be straight: `BP_Shelf_Round` draws its rows as circles.

---

## The storage component

`BP_StorageComponent` collects the rows on its actor and answers the questions the rest of the game asks:

| Function | Answers |
|---|---|
| `HasRoomFor(Product)` | Is there space for one more of these |
| `HasUnitOf(Product)` | Is there at least one of these here |
| `StockedProducts()` | What is being held right now |
| `AddUnit` / `RemoveUnit` | Put one in, take one out |

It also draws the units, and it rebuilds that display from the rows whenever the stock changes.

---

## The shelf

`BP_ShelfBase` is the actor. It adds four fields on top of the storage component:

| Field | What it does |
|---|---|
| `ShelfMesh` | The mesh |
| `ShelfType` | A `DA_ShelfType_*`, a list of accepted categories |
| `InitialProduct` | What it is stocked with when the level starts |
| `InitialUnits` | How many |

A shelf refuses a product whose `Category` is not in its `ShelfType.AcceptedCategories`, before the unit leaves the box.

Three ship with the template:

| Blueprint | Shape |
|---|---|
| `BP_Shelf_Gondola100` | Five straight rows: a base and four shelves |
| `BP_Shelf_Round` | A three-tier round display, rows drawn as circles |
| `BP_Shelf_FridgeTwoDoor` | Ten rows over two compartments, behind two glass doors |

To build your own, see [Build your own shelf](make_your_own_shelf.md).

---

## Filling and emptying

With a box in your hands, look at a shelf:

| Input | What it does |
|---|---|
| **Left mouse** | Move one unit from the box to the shelf |
| **Right mouse** | Take one unit back off the shelf |

The unit flies between the two on a short arc. Bare-handed, the same two gestures move one unit at a time with no box involved.

---

## Locking a shelf behind a condition

Override `IsAccessible()` on a child of `BP_ShelfBase` and return your own state. Both gestures and both prompt lines disappear together while it returns `false`.

`BP_Shelf_FridgeTwoDoor` is the shipped example: it returns `DoorsOpen`, so a fridge with its doors shut cannot be stocked from.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
