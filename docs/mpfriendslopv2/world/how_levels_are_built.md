# How levels are built

A level is a floor plan made of modular meshes on one grid, lit only by lamps, and given its look by two actors: a fog and a post-process volume. The start room of `L_Procedural` and the rooms in `Maps/Modules/` are built that way. Open them when you want to see a rule in practice.

- The kit: `Content/MPFriendslop/Meshes/Environments/Modular/`
- The master material: `Content/MPFriendslop/Materials/Props/Master/M_Prop_Master`

This page is the mental model. The pages after it are the recipes.

---

## The grid

| Measure | Value |
|---|---|
| One cell | `400` x `400` cm |
| Half step | `200` cm |
| Floor to ceiling | `300` cm |
| Floor and ceiling slab thickness | `16` cm |

Every module is placed at scale `1`. The player robot is about `180` cm to the head, so a `300` cm ceiling leaves a little over a metre above it.

---

## The modules

| Folder | Meshes |
|---|---|
| `Floors/` | `SM_Mod_Floor_400`, `SM_Mod_Floor_200`, `SM_Mod_Floor_Hatch_400`, `SM_Mod_Floor_Corner_400`, `SM_Mod_Floor_Edge_400`, `SM_Mod_Floor_Extraction_400` |
| `Walls/` | `SM_Mod_Wall_400`, `SM_Mod_Wall_200`, `SM_Mod_Wall_Window_400`, `SM_Mod_Wall_Pipes_400`, `SM_Mod_Wall_Damaged_400`, `SM_Mod_Wall_Doorway_400`, `SM_Mod_Wall_DoorwaySingle_400`, `SM_Mod_Wall_DoorPocket_200`, `SM_Mod_Wall_CornerInner_400`, `SM_Mod_Wall_CornerOuter_400`, `SM_Mod_Wall_Pillar_40` |
| `Ceilings/` | `SM_Mod_Ceiling_400` |
| `Openings/` | The door frames and leaves used by `BP_SlidingDoor_Single` and `BP_SlidingDoor_Double` |

`SM_Mod_Floor_Hatch_400` is a closed decorative hatch with no mechanism. The window pane of `SM_Mod_Wall_Window_400` is an opaque tinted panel that blocks the player on purpose.

Three props sit in `Meshes/Props/`: `SM_Barrel_01`, `SM_Shelf_01` and `SM_Trolley_01`. They are decor, with their pivot at the bottom centre. The shelf has no back panel, so either long face can go against a wall.

`L_Showcase` lays out the template's meshes, sorted by family. It is quicker than browsing folders when you look for a piece.

---

## Pivots, and where each piece goes

**Floors and ceilings** fill a cell. The slab covers X `0` to `400` and Y `-400` to `0` from its pivot, so the slab of cell `(i, j)` goes at `(400 x i, 400 x (j+1), 0)`, yaw `0`. The ceiling uses the same location with Z = `300`.

**Walls** sit on the edges of the grid, not in the cells. The pivot is at the foot of the wall, on its centre line, and the wall runs along +X, `300` cm high. An edge along X takes yaw `0`, an edge along Y takes yaw `90`. The wall is centred on the line, so it overlaps each neighbouring slab by about `12` cm. That is how the kit is meant to fit.

**Corners** replace two straight walls. `SM_Mod_Wall_CornerInner_400` and `SM_Mod_Wall_CornerOuter_400` carry two `400` cm branches and are placed on a grid node:

| Yaw | Branches go towards |
|---|---|
| `0` | +X and -Y |
| `90` | +Y and +X |
| `180` | -X and +Y |
| `270` | -Y and -X |

Use `Inner` when the room is inside the angle, `Outer` otherwise. When a node cannot take a corner, because one branch is already a doorway, a window or a decorated wall, close it with `SM_Mod_Wall_Pillar_40`, centred on the node. Without it, two straight walls leave a small notch on the outside of the angle.

The pipes of `SM_Mod_Wall_Pipes_400` and the damage of `SM_Mod_Wall_Damaged_400` are on the local +Y face only. Check which side faces the room before you place them.

---

## Why the level is dark without lamps

The project renders without global illumination and without baked lighting:

| Project setting | Value | What it means for your level |
|---|---|---|
| `Dynamic Global Illumination Method` | `None` | No bounce light. A room without a lamp is black |
| `Reflection Method` | `None` | Reflections come from reflection captures only |
| `Allow Static Lighting` | off | Nothing is baked. Every light is Movable |
| `Shadow Map Method` | `Virtual Shadow Maps` | Clean shadows on curved meshes without tuning each light |

`L_Procedural` and its rooms have no sun and no sky light. Every bit of light in it comes from the lamps of [Place, switch and make lamps](lamps_and_lights.md).

---

## The two actors that give a level its look

| Actor | What it carries | In `L_Procedural` |
|---|---|---|
| `ExponentialHeightFog` | The haze that swallows the far end of a corridor | `Fog Density` `0.9`, `Fog Height Falloff` `0.001` |
| `PostProcessVolume`, with `Infinite Extent (Unbound)` ticked | The locked exposure, the grade, and `MI_InteractableOutline` in its `Post Process Materials` | `Min EV100` and `Max EV100` both at `5` |

The exposure is locked so the picture does not drift when you walk from a lit room into a dark one. A higher EV100 value makes the whole level darker.

When a map is too bright or too dark, change the exposure first, then `Fog Density`. They act on the whole level in one field each.

!!! warning
    `MI_InteractableOutline` in the volume's `Post Process Materials` is what draws the outline on interactable objects. A level without it shows no outline at all, and nothing reports it.

---

## Recolouring the level

The kit, the props and the loot are painted with Material Instances of `M_Prop_Master`. Glass is the exception: the window pane and the glass vase use their own glass material. Stripes, floor markings and dots are drawn by the shader from the mesh's own position, so they follow a module you move, rotate or duplicate. There are no textures to repaint.

| Instance | Used on | Parameters worth changing |
|---|---|---|
| `MI_SteelPainted` | Walls | `BaseColor`, `StripeColor`, `StripeHeight`, `StripeWidth`, `StripeEnabled` |
| `MI_SteelPainted_Floor` | Floors | `BaseColor`, `MarkingColor`, `MarkingWidth`, `DotsEnabled`, `DotColor` |
| `MI_SteelPainted_FloorCorner`, `MI_SteelPainted_FloorEdge`, `MI_SteelPainted_FloorHatch` | The floor variants | Same as the floor |
| `MI_SteelPainted_DoorFrame` | Door frames, and the ceiling slab | `BaseColor` |

The instances live in `Materials/Environments/Modular/`. Change one and every module that uses it changes with it, with no graph to open. Set `StripeEnabled` to `0` to remove the wall stripe.

The ceiling slab uses `MI_SteelPainted_DoorFrame`, which is why the ceilings are dark. For another ceiling colour, give that slot on `SM_Mod_Ceiling_400` another instance: one change covers every slab.

---

## Where to go next

- [Build your own level](build_your_own_level.md)
- [Place, switch and make lamps](lamps_and_lights.md)
- [Sounds, footsteps and surfaces](sounds_and_footsteps.md)

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
