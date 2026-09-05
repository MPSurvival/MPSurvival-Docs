# The mesh kit

Every mesh in the template was built in Blender, from primitives, by scripts we own. No image was imported and no model was bought, so the kit redistributes with no question about where it came from.

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

The note, the coin and the card are the only textured meshes. Everything else is coloured by its material.

---

## The modular grid

| Constant | Value |
|---|---|
| Module | 400 cm |
| Storey | 300 cm |
| Wall thickness | 30 cm |

Module pivots land on the grid, so pieces snap together without measuring.

**When you lay out a room**, line the **inside faces** of the walls up with the placement grid, which snaps to multiples of 50 from the world origin. Walls are 30 thick and centred on the module lines, so their inside faces would otherwise land at module +/- 15 and the placement cell against every wall would be half inside it. In `L_ExampleMap` the walls are moved back 15 cm for exactly that reason.

---

## Material slots are semantic

**The name of a material slot is the assignment key.** A mesh comes out of Blender with slots named for what the surface is, not for what it looks like:

`M_Wood` - `M_Metal` - `M_SteelPainted` - `M_SteelWhite` - `M_Plastic` - `M_Glass` - `M_Cardboard` - `M_Fabric` - `M_Paper` - `M_Label` - `M_Concrete` - `M_Tile` - `M_Rubber` - `M_Asphalt` - `M_RoadPaint` - `M_Bark` - `M_Leaf` - `M_Screen` - `M_Bronze`

On the Unreal side there is one master, `M_PropsMaster`, and one instance per slot, so assigning materials to an imported mesh is mechanical. Never assign the master itself to a mesh: a new variant should be a 2 kB instance, not a shader compile.

---

## Two flags that will bite you

**`Used with Instanced Static Meshes`.** Shelf and box contents are drawn with instanced static mesh components, and a material without that flag is silently replaced by the grey checker there, with nothing in the log and a perfectly correct thumbnail. If your products come out as grey checkerboards, this is it: set the flag and re-save the material.

**`Is Dynamic Obstacle`, on the asset.** Anything that moves to open a passage, a door leaf or a gate, needs it ticked on the **static mesh**, not on the component.

---

## Collision

Every mesh that needs proper collision ships with a `UCX_` hull. Auto-generated hulls are a bad fit here, because shop fittings are full of holes: a convex hull over a gondola fills in the shelves, and customers then refuse to approach it.

Two rules when you draw your own:

- **A hull for a moving part is a swept volume**, not the part's silhouette. A fan blade's hull covers the disc it sweeps.
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

Measure the real thing. Everything in this kit was modelled from photographs and spec sheets: an object modelled from memory comes out plausible and wrong, and you only see it in game once the room is full of them.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
