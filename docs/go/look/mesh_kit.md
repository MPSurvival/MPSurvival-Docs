# The mesh kit

Every mesh in the template was built by us, in Blender, from primitives, by scripts we own. No image was imported and no model was bought, which is what makes the kit redistributable without any question about where it came from.

84 static meshes and 8 skeletal ones, all low poly.

---

## What is in it

| Family | Contents |
|---|---|
| Modular architecture | Walls, floors, ceilings, ramps, thresholds, a ceiling grid and a service panel |
| Openings | Door frames, a solid door, a glass door, a storefront window and a storefront door |
| Street | Road, pavement, driveway, four building facades, a street lamp, a tree and a tree grate |
| Shop fittings | Gondola, round display, two-door fridge, storage rack, checkout counter |
| Checkout | POS display, card terminal, barcode scanner, cash drawer, four note stacks, four coin trays, a loose note, a loose coin, a payment card, a shopping bag |
| Delivery | Three box sizes, each as a body and two flaps |
| Props | Trash bin, office desk, gift box, gumball machine, ceiling fan, desk fan, light switch, picture frames, planters, desk plant, the OPEN sign, the ad easel |
| Products | Cereal, canned soup, soda can, shampoo, detergent, batteries |
| Characters | The mannequin, six hair styles and a clerk's vest |

The three payment objects, the note, the coin and the card, are the only textured meshes in the kit. Everything else is coloured by its material.

---

## The modular grid

Architecture is built on one grid, and it does not get re-decided per piece:

| Constant | Value |
|---|---|
| Module | 400 cm |
| Storey | 300 cm |
| Wall thickness | 30 cm |

A module pivot lands on the grid, which is what lets pieces be snapped together without measuring.

One thing to know if you lay out a room: the walls are 30 thick and **centred** on the module lines, so their inside faces land at module ± 15. The placement grid, meanwhile, snaps to multiples of 50 from the world origin. Line the inside faces up with the placement grid, not the module lines, or the cell against every wall is half inside it. In `L_ExampleMap` the walls were moved back 15 cm for exactly this reason.

Four corner posts of 30 × 30 are missing from the outside of the shipped shell, which is arithmetic rather than an oversight: a ring of ten 400 modules covers an interior of (1200 − 30) × (800 − 30), and a full 1200 × 800 interior needs a little more than the ring provides. They are hidden inside neighbouring facades and behind the building.

---

## Material slots are semantic

**The name of a material slot is the assignment key.** A mesh comes out of Blender with slots named for what the surface is, not for what it looks like:

`M_Wood` · `M_Metal` · `M_SteelPainted` · `M_SteelWhite` · `M_Plastic` · `M_Glass` · `M_Cardboard` · `M_Fabric` · `M_Paper` · `M_Label` · `M_Concrete` · `M_Tile` · `M_Rubber` · `M_Asphalt` · `M_RoadPaint` · `M_Bark` · `M_Leaf` · `M_Screen` · `M_Bronze`

On the Unreal side there is a single master, `M_PropsMaster`, and one instance per slot. Assigning materials to an imported mesh becomes mechanical rather than a decision per mesh.

A master is never assigned directly to a mesh. The master carries the parameters, the instance carries the values, and a new variant costs a 2 kB asset instead of a shader compile.

---

## Two flags that will bite you

**`Used with Instanced Static Meshes`.** Shelf and box contents are drawn with instanced static mesh components. A material without that flag is silently replaced by the default grey checker on an ISM, with nothing in the log and a perfectly correct asset thumbnail. If products come out as grey checkerboards, this is it. Set the flag and re-save the material.

**`Is Dynamic Obstacle`, on the asset.** Anything that moves to open a passage, a door leaf or a gate, needs it ticked on the **static mesh**, not on the component.

---

## Collision

Every mesh that needs proper collision ships with a `UCX_` hull. Auto-generated hulls are a bad fit here because shop fittings are full of holes: a convex hull over a gondola fills in the shelves and customers refuse to approach it.

Two rules that came out of building the kit:

- **A hull for a moving part is a swept volume**, not the part's silhouette. A fan blade's hull has to cover the disc it sweeps.
- **A bounding box lies as soon as the object is hollow.** A drawer's hull has to reach the bottom of the coin trays, or a trace aimed at the money stops before it gets there.

---

## Adding your own meshes

Nothing forces you to use the kit. What the game expects from a mesh is short:

| Requirement | Why |
|---|---|
| Pivot on the floor, centred in the footprint | Placement snaps the pivot |
| A `UCX_` hull | Auto hulls fill in shelf openings |
| Semantic slot names | So the master material assignment stays mechanical |
| Real-world dimensions in centimetres | Footprints and grids are all in cm |

Measure the real thing. Everything in this kit was modelled from photographs and spec sheets, with the dimensions and their sources written down, because an object modelled from memory comes out plausible and wrong, and you only see it in game once the room is already full of them.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
