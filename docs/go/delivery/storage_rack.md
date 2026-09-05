# The back room rack

`BP_StorageRack` stores **whole boxes**, not units. Carry a box, look at the rack, press `E`, and it goes on the first free slot. Look at a box on the rack and press `E` to take it back, still holding whatever was in it.

*Store* only appears in the carry panel when there is a free slot, so the prompt never lies about what the key will do.

---

## The slots

There is no slot list to fill in. The positions come from three fields:

| Field | What it does | Shipped |
|---|---|---|
| `LevelHeightsCm` | The height of each deck | `13.3`, `57.3`, `101.3`, `145.3` |
| `SlotsPerLevel` | Slots across a deck | `3` |
| `SlotProbeCm` | Radius used to test whether a slot is taken | `8` |

Four decks times three slots is twelve boxes. **To make a bigger rack**, add an entry to `LevelHeightsCm` or raise `SlotsPerLevel`, and give it the mesh to match.

The rack fills left to right and bottom to top, stopping at the first gap. A stored box has its physics turned off so the next one sliding in cannot push it off the deck, and turned back on when you pick it up.

---

## What it does not do

The rack holds no products of its own. It cannot be restocked from, customers never take anything off it, and employees do not unpack from it. It is a place to put boxes.

---

## Placing one by hand

`DA_Structure_StorageRack` exists, but a rack you drop in a level only becomes editable and sellable once you set `Structure` on its `BP_PlaceableComponent` to that asset. Leave it empty and edit mode will not pick the rack up. See [Add a placeable structure](../build/add_a_structure.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
