# How placement works

Buying furniture and placing it are two separate things. You buy from the **Structures** app on the office computer, which puts the item in your stock, and you place it later with the build key, anywhere you like. That split is what stops you walking back to the office for every shelf.

---

## The two components

| Component | Where | What it does |
|---|---|---|
| `BP_PlacementManager` | On `BP_StoreCharacter` | Aims, validates, places, and runs edit mode |
| `BP_PlaceableComponent` | On anything placeable | Marks the actor as furniture and remembers which `DA_Structure_*` it came from |

The manager lives on the character rather than the controller because the aiming trace needs the camera, which is on the character, next to the interaction, carry and view components.

Anything carrying a `BP_PlaceableComponent` with a valid `Structure` can be picked back up, including furniture you placed by hand in the editor. If `Structure` is empty on an actor you dropped in a level, edit mode will not see it.

---

## Floors, walls and ceilings

Placement is not a floor grid. `TraceFoundation` returns the **normal** of whatever surface you are aiming at, snapping happens in the plane of that surface, and the ghost orients itself so its local +Z is the surface normal.

That is what makes wall lamps, ceiling fans and desk clutter possible without a second system.

**A surface is only a foundation if it says so.** There is a dedicated trace channel, `StructureFoundation`, which defaults to **Ignore**. Floors, walls, ceilings and desk tops block it. A gondola does not, which is why you cannot balance a gift box on a shelf.

Making something a foundation is one line in its Collision panel. There is no graph to open.

Three booleans on the structure decide where it may go:

| Field | Meaning |
|---|---|
| `CanPlaceOnFloor` | Any horizontal surface facing up, which includes desk and counter tops |
| `CanPlaceOnWall` | Vertical surfaces |
| `CanPlaceOnCeiling` | Horizontal surfaces facing down |

They are three booleans rather than one enum on purpose. A wall lamp can legitimately go on a wall **and** a ceiling.

The ghost only tilts onto a surface it is allowed on. Aim a desk at a wall and it stays flat and red, instead of flattening itself against the wall and then telling you it is invalid.

---

## Snapping

The snap step is on the **structure**, not on the manager:

| Field | What it does |
|---|---|
| `SnapStepCm` | The grid step, in centimetres |
| `FootprintCells` | The footprint, in cells |
| `RotationStep` | The rotation increment |

Physical size is `FootprintCells × SnapStepCm`. A gondola snaps at half a metre, a desk fan at ten centimetres, and a 125 cm fridge works because its step is 25.

`R` rotates a quarter turn around the surface normal. The footprint rotates with the structure, so a 1×3 turned sideways occupies 3×1 for both the snap and the overlap test.

---

## Refusals are named

`ValidatePlacement` returns an `S_PlacementResult`, and the `Reason` field is a `Text` shown under the crosshair. Nothing is ever refused in silence.

| Reason | Meaning |
|---|---|
| Nothing selected to build | No structure is armed |
| Not allowed in this area | You are outside every zone the structure allows |
| A customer is standing here | Someone is in the footprint |
| Something is already here | Another placed structure overlaps |

The four messages are variables on the manager, so translating or rewording them means editing fields.

The overlap test only considers placed structures and customers. It does not test the footprint against walls, so a shelf can still bite a few centimetres into a wall if the grid and the wall do not line up. In the shipped level they do line up: every interior wall face sits on a grid multiple, so the full cell against each wall is free.

---

## The build menu

`B` opens a full-screen menu: a grid of tiles on the left, a detail card on the right, category tabs across the top. Tabs with nothing in them are hidden.

Clicking a tile arms that structure **and closes the menu**, so you go straight back to aiming. `B` again leaves build mode entirely.

`B` is a two-state toggle and nothing else. Changing structure without leaving costs two presses, which is the price of a key that always does the same thing.

There is no money in this menu. You do not buy here; the card shows the footprint and how many you have in stock.

---

## Edit mode

Edit mode is a sub-state of build mode, not a third mode. The `EDIT` button at the bottom right of the build menu enters it, and `EDIT MODE` appears under the crosshair with `[B] Exit`.

Aim at a placed structure and the ghost snaps onto it. `X` picks it up and returns it to your stock for free, keeping its rotation, so putting it back down puts it back the way it was.

Some things refuse to be picked up, and they say why:

| Refusal | When |
|---|---|
| Customers are at this checkout | Its queue is not empty |
| An employee is working here | It has a cashier |
| This shelf still has stock on it | It is not empty |

Ordinary interaction is suspended for as long as you are in build or edit mode, so you cannot accidentally open a fridge while placing one.

---

## Buying and selling

The **Structures** app on the office computer is the shop. `BuyStructure` refuses if you cannot afford it and writes a `Purchase` transaction. `SellStructure` pays back `RefundRatio`, half by default.

Stock lives in `OwnedStructures` on `BP_StoreManager`, and only that component's own two functions touch the array.

One thing to be careful about, since the shipped level lets you sell the furniture you started with: **you can sell your last checkout.** With no till, nothing generates income, and buying one back costs twice what selling it paid. There is no guard against it. Refusing a sale when `stock + placed == 1` is one condition in `SellStructure` if you want one.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
