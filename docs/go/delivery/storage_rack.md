# The back room rack

`BP_StorageRack` stores **whole boxes**, not units. That is the point of it, and it is what makes it different from a shelf.

You carry a box, look at the rack, press `E`, and the box goes on the first free slot. To get it back you look at the box on the rack and press `E`. It is still the same box, still holding whatever was in it.

---

## How the slots work

The rack has no slot list to fill in. Its positions are computed from two fields:

| Field | What it does | Shipped default |
|---|---|---|
| `LevelHeightsCm` | The height of each deck | `13.3`, `57.3`, `101.3`, `145.3` |
| `SlotsPerLevel` | Slots across a deck | `3` |
| `SlotProbeCm` | Radius used to test whether a slot is taken | `8` |

Four decks times three slots is twelve boxes. Making the rack longer is one more entry in `LevelHeightsCm` or a higher `SlotsPerLevel`, and the mesh to go with it.

Whether a slot is free is **read from the world**, not stored. `IsSlotFree` does a sphere overlap filtered to delivery boxes, ignoring the one you are carrying. Nothing has to be kept in sync, and a box that ends up on the rack some other way is seen straight away.

`FindFreeSlotFor` returns the first free slot, so the rack fills left to right and bottom to top and stops at the first gap.

---

## A stored box is frozen

When a box is placed on the rack, its physics simulation is turned off. Without that, walking past it or sliding the next box in would push it off the deck.

It is turned back on when you pick it up. Two writers on the same flag, ordered by the gesture: you cannot store a box you are not holding, and you cannot hold a box that is on the rack.

---

## Taking a box back needs no code

The stored box keeps its own `BP_InteractableComponent`. Look at it, press `E`, and `PickUp` puts it back in your hands.

That falls out for free because the rack never takes ownership of the box. It puts it down in a place and forgets about it.

---

## The action line

*Store* appears in the carry panel only when you are looking at a rack **and** there is a free slot. `CanStoreInto` checks both, so the prompt never lies about what the key will do.

---

## What it is not

The rack holds no products and no storage component. It cannot be restocked from, a customer will never take anything off it, and an employee will not unpack from it. It is a place to put boxes.

One thing to check before you place one by hand: `DA_Structure_StorageRack` exists, but a rack dropped in a level only becomes editable and sellable once you set `Structure` on its `BP_PlaceableComponent` to that asset. Leave it empty and edit mode will not pick the rack up. See [Add a placeable structure](../build/add_a_structure.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
