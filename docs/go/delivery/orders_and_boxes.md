# From an order to a box on the floor

You order cases in the Market app. Hours later, boxes land on the delivery bay. Nothing in between is faked: the box that arrives is a real actor, sized to the product, holding the right number of units, with the product's icon printed on its docket.

---

## The chain

```
Market app  →  BP_DeliveryManager.QueueOrder  →  S_PendingOrder
                                                     ↓  OnHourChanged
                                              DeliverDueOrders
                                                     ↓
                          BP_DeliveryBay.TakeNextSlot  →  a box is spawned
```

`BP_DeliveryManager` is a component on the game state, and it is the only owner of pending orders. It subscribes to the store clock's `OnHourChanged` rather than running a timer, so nothing ticks and orders cannot arrive while the clock is frozen.

An `S_PendingOrder` holds the product, the number of cases, and the arrival day and hour. The arrival time is computed by the clock's owner, `BP_StoreManager.ClockAfterHours`, so there is only one calendar in the project.

---

## When a box arrives

Two fields on the product decide, and they are on the product because delivery speed is a property of what you are buying:

| Field | Effect |
|---|---|
| `DeliveryDelayHours` | The box arrives that many game hours after the order |
| `DeliversNextDay` | The box arrives at opening time the next day, whatever the hour you ordered |

`DeliversNextDay` wins when both are set. Ordering at 20:00 with next-day delivery gets you a box at 08:00 tomorrow, and ordering at 06:00 gets you one at 08:00 today, because "the next opening" is computed rather than assumed to be tomorrow.

---

## Which box you get

You do not pick a box size. `PickBoxFor` picks the **smallest** box whose capacity covers `UnitsPerCase` for that product, and falls back to the largest one if nothing fits.

Capacity is derived from the two Data Assets without spawning anything:

```
floor( InteriorSizeCm.X / FootprintCm.X )
    × floor( InteriorSizeCm.Y / FootprintCm.Y )
    × min( StackMax , floor( InteriorSizeCm.Z / StackHeightCm ) )
```

The list of box sizes the manager can choose from is `BoxSizes` in its Details panel. Adding a size is [one Data Asset and one child Blueprint](add_a_box_size.md).

If a case is bigger than the biggest box, the extra units are silently refused when the box is filled. Either lower `UnitsPerCase` or add a bigger box.

---

## The delivery bay

`BP_DeliveryBay` is the actor that says where boxes land. Drop one in your level and it registers itself with the manager at `BeginPlay`, so the manager never goes looking for it.

| Field | What it does | Shipped default |
|---|---|---|
| `SlotPitchCm` | Spacing between two drop slots | `70` |
| `SlotsPerRow` | Slots across | `4` |
| `SlotRows` | Rows deep | `3` |
| `DropHeightCm` | How far above the floor a box appears | `20` |

`TakeNextSlot` hands out the next slot and wraps around after `SlotsPerRow × SlotRows`, so a very large delivery starts stacking on top of the first slots rather than marching off into the street.

The bay has no mesh. Nothing in the world tells the player where boxes will fall, so put it somewhere the level makes obvious, like a roller door or a painted rectangle on the floor.

---

## What is inside a box

A box is a storage row, the same one shelves use. It works out its own interior layout from the product's footprint and the box's interior dimensions, and shows one instanced mesh per unit, aligned to the inside corner rather than to the mesh origin.

| Field on the box | What it does |
|---|---|
| `Size` | A `DA_Box_*`. Everything about the shape comes from here |
| `Product` | What is inside |
| `InitialUnits` | How many at spawn |

All three are **Expose on Spawn**, which is why a delivery does not need the `_Small` / `_Medium` / `_Large` child Blueprints. The manager spawns the base class with the right values and the construction script builds the right box.

The three children exist for boxes you place by hand in a level.

Opening a box swings both flaps on a timeline, `150` degrees, and the contents are hidden until it is open. Press `F` to open the box you are carrying, or the one you are looking at if your hands are empty.

The docket on the front flap is a real widget in the world showing the product's icon, painted at actual size, and it is read from above because that is how you look at a box on the floor.

---

## Emptying it

Carry the box, look at a shelf, and press left mouse to move one unit across. Right mouse takes one back. See [How shelves and storage work](../stock/how_storage_works.md).

The box has to be open, or you would watch a bottle pass through closed cardboard.

When you are done with a box, look at the trash bin with it in your hands and press `E`. The bin destroys it. Only boxes go in the bin, so a barcode scanner in your hands does nothing.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
