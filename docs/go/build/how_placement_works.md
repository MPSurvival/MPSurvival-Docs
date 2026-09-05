# How placement works

Buying furniture and placing it are two separate things. You buy from the **Structures** app on the office computer, which puts the item in your stock, then you place it later with `B`, anywhere you like.

---

## The keys

| Key | What it does |
|---|---|
| `B` | Open the build menu, and leave build mode |
| Left mouse | Place the armed structure |
| `R` | Rotate a quarter turn |
| `X` | In edit mode, pick a placed structure back up |

Clicking a tile in the menu arms that structure and closes the menu, so you go straight back to aiming. Ordinary interaction is suspended while you are in build mode, so you cannot open a fridge while placing one.

The menu shows no prices: you are not buying here, and the card shows the footprint and how many you have in stock.

---

## Floors, walls and ceilings

Placement follows the surface you aim at, so wall lamps and ceiling fans work with no second system. Three booleans on the structure decide where it may go:

| Field | Meaning |
|---|---|
| `CanPlaceOnFloor` | Any horizontal surface facing up, which includes desk and counter tops |
| `CanPlaceOnWall` | Vertical surfaces |
| `CanPlaceOnCeiling` | Horizontal surfaces facing down |

They are three booleans rather than one choice, because a wall lamp can legitimately go on a wall **and** a ceiling.

**A surface only accepts furniture if it blocks the `StructureFoundation` trace channel.** Floors, walls, ceilings and desk tops block it; a gondola does not, which is why you cannot balance a gift box on a shelf. Making your own mesh a foundation is one line in its Collision panel.

---

## Snapping

The snap settings are on the **structure**, so a gondola and a desk fan can use different grids:

| Field | What it does |
|---|---|
| `SnapStepCm` | The grid step, in centimetres |
| `FootprintCells` | The footprint, in cells |
| `RotationStep` | The rotation increment |

Physical size is `FootprintCells × SnapStepCm`: a gondola snaps at half a metre, a desk fan at ten centimetres, and a 125 cm fridge works because its step is 25. The footprint rotates with the structure, so a 1×3 turned sideways occupies 3×1.

---

## When placement refuses

The reason is written under the crosshair, always:

| Reason | Meaning |
|---|---|
| Nothing selected to build | No structure is armed |
| Not allowed in this area | You are outside every zone the structure allows |
| A customer is standing here | Someone is in the footprint |
| Something is already here | Another placed structure overlaps |

The four messages are variables on `BP_PlacementManager`, so rewording them means editing fields.

The overlap test looks at placed structures and customers, not at walls, so a shelf can bite a few centimetres into a wall if your grid and your walls do not line up. In the shipped level they do: every interior wall face sits on a grid multiple.

---

## Edit mode

The `EDIT` button at the bottom right of the build menu enters it, and `EDIT MODE` appears under the crosshair.

Aim at a placed structure and the ghost snaps onto it. `X` picks it up and returns it to your stock for free, keeping its rotation, so putting it back down puts it back the way it was.

Some things refuse, and say why:

| Refusal | When |
|---|---|
| Customers are at this checkout | Its queue is not empty |
| An employee is working here | It has a cashier |
| This shelf still has stock on it | It is not empty |

---

## Making an actor placeable

Add a `BP_PlaceableComponent` and set its `Structure` to the `DA_Structure_*` it came from. That is what makes it pickable in edit mode and sellable — including furniture you dropped into the level by hand in the editor. Leave `Structure` empty and edit mode will not see it.

See [Add a placeable structure](add_a_structure.md).

---

## Buying and selling

The **Structures** app is the shop. Selling pays back `RefundRatio`, half by default.

Careful with one thing, since the shipped level lets you sell the furniture you started with: **selling your last checkout is allowed**, and with no till nothing generates income, while buying one back costs twice what selling it paid. To hold that one back, refusing a sale when `stock + placed == 1` is one condition in `SellStructure`.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
