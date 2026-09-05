# How shelves and storage work

Every place a product can sit uses the same two components: a **storage component** that arbitrates, and one or more **storage rows** that hold things. A gondola, a round display, a fridge and the inside of a cardboard box are all the same machinery with different rows drawn on them.

This page is the mental model. The pages after it are the recipes.

---

## A row is a curve, not a grid

`BP_StorageRow` is a child of `SplineComponent`. You draw it along the front edge of a shelf in the viewport, and it works out on its own how many units fit.

```
along  =  SplineLength / product width
deep   =  DepthCm      / product depth
up     =  min( StackMax , ClearanceCm / StackHeightCm )
```

Nothing about the layout is written down anywhere. Change which product goes on the shelf and the row re-slices itself.

| Field | What it does |
|---|---|
| `DepthCm` | How far back from the curve the row extends |
| `ClearanceCm` | The free height above the row |

The template used to use a rectangular 2D grid here, and it was replaced for one reason: a grid forces the furniture to be a rectangle. With a curve you can build a round display, an L-shaped bin or a slanted magazine rack without touching the component.

**Two rules follow from that, and they are the only two you have to remember:**

1. **Draw the spline along the front edge of the shelf, left to right, and depth pushes toward the spline's Right vector.** A row drawn backwards fills toward the front of the shelf and the products hang in the air. It is the one trap in the system, and it is obvious the first time you see it in game.
2. **`ClearanceCm` is a real constraint.** A 34 cm cereal box will not go on a shelf with 35 cm of pitch once you account for the shelf board, but it goes on the top row where there is nothing above it. That is a real shop behaving like a real shop, not a bug.

---

## The storage component

`BP_StorageComponent` is a child of `SceneComponent`. At `BeginPlay` it collects every `BP_StorageRow` on its actor, and from then on it is the thing that answers questions:

| Function | Answers |
|---|---|
| `HasRoomFor(Product)` | Is there space for one more of these |
| `HasUnitOf(Product)` | Is there at least one of these on me |
| `StockedProducts()` | Which products am I actually holding right now |
| `AddUnit` / `RemoveUnit` | Put one in, take one out |

It also owns the display. There is one `InstancedStaticMeshComponent` per product being stored, and one instance per unit. The rows are the single source of truth: `RefreshDisplay` throws the instances away and rebuilds them from the rows, so there is no parallel list to keep in sync and no way for the visuals to drift from the stock.

The component knows nothing about products, shelf types or the shop. That is what lets the delivery box reuse it without inheriting anything from a shelf.

---

## The shelf on top

`BP_ShelfBase` is the actor. It adds four things to the storage component:

| Field | What it does |
|---|---|
| `ShelfMesh` | The mesh, applied in the construction script |
| `ShelfType` | A `DA_ShelfType_*`, which is nothing but a list of accepted categories |
| `InitialProduct` | What the shelf is stocked with when the level starts |
| `InitialUnits` | How many |

The category rule lives on the shelf, not in the storage component. A shelf refuses a product whose `Category` is not in its `ShelfType.AcceptedCategories`, and the refusal happens before the unit ever leaves the box.

Three children ship with the template:

| Blueprint | Shape |
|---|---|
| `BP_Shelf_Gondola100` | Five straight rows: a base and four shelves |
| `BP_Shelf_Round` | A three-tier round display, rows drawn as circles |
| `BP_Shelf_FridgeTwoDoor` | Ten rows, five levels across two compartments, behind two glass doors |

---

## Filling and emptying

With a box in your hands, look at a shelf and press **left mouse** to move one unit from the box to the shelf, or **right mouse** to take one back. The unit flies between the two on a short arc.

That flight is `BP_ProductFlight`, and it deserves a mention because it is reused everywhere: the scanned item flying into the bag at the checkout, the payment card flying to the terminal. It knows nothing about shops. It takes a mesh, a start point, a **target component** with a local offset, an arc height and a duration, and broadcasts `OnArrived`.

Targeting a component rather than a world point matters. The box is in your hands and it moves during the flight, so the unit lands in the box even if you turn around while it is in the air.

The unit is removed from the source at the click and added to the target on arrival, never both and never neither. One transfer per box can be in the air at a time.

Bare-handed, `PlaceOne` and `TakeOne` do the same thing with no box involved.

---

## Locking a shelf behind a condition

`BP_ShelfBase.IsAccessible()` returns `true`. Override it on a child and both gestures and both action lines disappear together, because the box asks that one function before it offers to stock or take back.

The fridge is the shipped example: it overrides `IsAccessible` to return `DoorsOpen`, so a closed fridge cannot be stocked from, and the panel next to your box does not offer to.

A closed fridge still *has* room and still *has* stock, so `HasRoomFor` and `HasUnitOf` are left alone. Availability and accessibility are two different questions.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
