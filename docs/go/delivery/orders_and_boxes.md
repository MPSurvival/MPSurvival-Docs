# From an order to a box on the floor

You order cases in the Market app on the office computer. Hours later, boxes land on the delivery bay: real actors, sized to the product, holding the right number of units, with the product's icon on the docket.

Orders arrive on the store clock, so nothing is delivered while the clock is frozen.

---

## When a box arrives

Two fields on the product decide:

| Field | Effect |
|---|---|
| `DeliveryDelayHours` | The box arrives that many game hours after the order |
| `DeliversNextDay` | The box arrives at the next opening time, whatever the hour you ordered |

`DeliversNextDay` wins when both are set. Order at 20:00 and the box is there at 08:00 tomorrow; order at 06:00 and it is there at 08:00 today.

---

## Which box you get

You do not pick a box size. The delivery takes the **smallest** box that holds one case of that product, from the `BoxSizes` list on `BP_DeliveryManager` in the game state's Details panel.

How many units fit is worked out from the interior size of the box and the `FootprintCm`, `StackHeightCm` and `StackMax` of the product. If a case is bigger than the biggest box, the units that do not fit are dropped: lower `UnitsPerCase` on the product, or [add a bigger box](add_a_box_size.md).

---

## The delivery bay

`BP_DeliveryBay` says where boxes land. Drop one in your level and deliveries find it on their own.

| Field | What it does | Shipped |
|---|---|---|
| `SlotPitchCm` | Spacing between two drop slots | `70` |
| `SlotsPerRow` | Slots across | `4` |
| `SlotRows` | Rows deep | `3` |
| `DropHeightCm` | How far above the floor a box appears | `20` |

Twelve slots at the shipped values, and a delivery larger than that starts again on the first slot rather than marching off into the street.

The bay has no mesh of its own, so put it somewhere the level makes obvious: a roller door, or a painted rectangle on the floor.

---

## The box

| Field | What it does |
|---|---|
| `Size` | A `DA_Box_*`. The shape comes entirely from here |
| `Product` | What is inside |
| `InitialUnits` | How many at spawn |

All three are Expose on Spawn, so a delivery spawns the base class with the right values. The `_Small` / `_Medium` / `_Large` children are there for boxes you place by hand in a level.

Press `F` to open the box you are carrying, or the one you are looking at if your hands are empty. The contents are hidden until it is open.

---

## Emptying it

Carry an open box, look at a shelf, **left mouse** moves one unit across and **right mouse** takes one back. See [How shelves and storage work](../stock/how_storage_works.md).

To get rid of an empty box, carry it to the trash bin and press `E`. Only boxes go in the bin.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
