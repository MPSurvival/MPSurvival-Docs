# The bonus mesh kit

The template comes with its own environment art so you have something to build a level out of on day one. Eighty seven modular building pieces and twenty six plants, all made for this project.

No system depends on them. Build with them, mix them with your own meshes, or delete the folder. Three door Blueprints are the only things that point at a specific piece, and they are covered further down.

- The meshes: `Content/TheLastTemplate/Meshes/Environments/`
- Their materials: `Content/TheLastTemplate/Materials/Environments/Props/` and `Content/TheLastTemplate/Materials/Environments/Vegetation/`

Open `L_MeshesShowcaseMap` to see every piece laid out on the ground with nothing else in the level. It is the fastest way to pick a piece before you place it. See [Maps, game modes and how the game starts](../start/maps_and_startup_flow.md).

---

## The grid

Everything is cut to one grid, and the sizes are written in the names.

- A full piece spans **400 cm**. That is the `400` in `SM_Mod_Wall_Brick_400`.
- A half piece spans **200 cm**, written `Half_200`.
- One floor is **300 cm** high. Walls, stairs and ladders are all cut for that height. A warehouse is two floors stacked.

Pivots sit on the grid rather than in the middle of the bounding box, so pieces line up by position alone. Turn grid snapping on, set it to 100 or 50, and drag. Nothing needs nudging.

---

## Outside pieces

`Content/TheLastTemplate/Meshes/Environments/Modular/`

| Folder | Pieces | What is in it |
|---|---|---|
| `Walls` | 15 | Brick, concrete and corrugated steel walls, in full and half lengths, with door, window and gate variants, a broken brick wall and a brick corner |
| `Floors` | 3 | A concrete slab, a broken slab, a steel grating |
| `Roofs` | 1 | A corrugated roof deck |
| `Structure` | 4 | An I beam column and an I beam girder, a concrete pillar, a roof truss |
| `Openings` | 4 | A door frame, a steel door leaf, a roller door, a steel window |
| `Traversal` | 4 | Steel stairs, a caged ladder, a catwalk, a railing |
| `Details` | 10 | A straight pipe and an elbow, a duct, an industrial lamp, a plank barricade, five rubble pieces |

---

## Inside pieces

`Content/TheLastTemplate/Meshes/Environments/Modular/Interior/`

| Folder | Pieces | What is in it |
|---|---|---|
| `Walls` | 13 | Plaster and wainscot walls with door, window, arch, half and damaged variants, plus bare stud walls |
| `Floors` | 5 | A wood floor and a broken one, exposed joists, a ceiling and a collapsed ceiling |
| `Openings` | 6 | A door frame, a door panel and a broken one, a boarded door, a sash window, a boarded window |
| `Traversal` | 4 | A wood stair flight and a broken one, a wood railing, a newel post |
| `Fixtures` | 3 | A base cabinet, a drawer cabinet, a shelf unit |

Interior pieces are prefixed `SM_Mod_Int_` and use the same 400 and 300 grid, so an interior wall drops straight into an exterior shell.

---

## Dock and quarantine

Two small sets of dressing, built to the same grid.

| Folder | Pieces | What is in it |
|---|---|---|
| `Dock` | 11 | A quay wall and a quay ladder, a bollard, a fender, a piling, a gangway, a gantry crane, a twenty foot container, a pallet crate, an oil drum, a chainlink fence |
| `Quarantine` | 4 | A Jersey barrier, a checkpoint booth, razor wire, a sign board |

The other folders under `Meshes/Environments/` are not part of the kit. `Water` holds the ocean plane, `Traps` the three tripwire pieces, `Props` the workbench and the training dummy, and `Menu` the window the main menu scene is built around.

---

## The openings, and the doors that already fit

Every wall variant cuts the same hole, so a door you build for one wall material fits all of them.

| Opening | Hole in the wall | Walls that carry it |
|---|---|---|
| Service door | 130 wide, 212 high | `SM_Mod_Wall_*_Door_400` |
| Loading gate | 240 wide, 222 high | `SM_Mod_Wall_Brick_Gate_400`, `SM_Mod_Wall_Corrugated_Gate_400` |
| Workshop window | 180 wide, 120 high, sill at 90 | `SM_Mod_Wall_*_Window_400` |
| Interior door | 92 wide, 210 high | `SM_Mod_Int_Wall_*_Door_400` |
| Interior arch | 160 wide, 215 high | `SM_Mod_Int_Wall_*_Arch_400` |

The collision leaves those holes open, so the player walks through a bay with nothing placed in it.

Three interactable doors ship already fitted to the kit. Drop one into a bay and it works.

| Blueprint | `Door Mesh Asset` |
|---|---|
| `BP_Interactable_Door_Steel` | `SM_Mod_Door_Steel` |
| `BP_Interactable_Door_Panel` | `SM_Mod_Int_Door_Panel` |
| `BP_Interactable_Door_Panel_Broken` | `SM_Mod_Int_Door_Panel_Broken` |

Those three fields are the only hard link between the kit and a gameplay Blueprint. If you delete the meshes, point `Door Mesh Asset` at your own leaf and the door graph carries on unchanged. See [Doors and the gate button](../inventory/doors_and_gate_button.md).

---

## Stairs, ladders and vaulting

Both stair flights, `SM_Mod_Stairs_Steel_400` and `SM_Mod_Int_Stairs_Wood_400`, climb one floor across one module. Place one against a 300 cm wall and its top lands on the next floor. `SM_Mod_Ladder_400` is one floor tall as well.

Vaulting is not a property of a mesh. To make an obstacle climbable, place a `BP_TraversableBlock`, put whatever mesh you want on its `Static Mesh` component, and draw the four ledge splines it carries. `Min Ledge Width`, which ships at `60`, is the width below which a ledge is refused. `BP_Traversable_Container` is the same actor with a container already on it.

For which move plays on which obstacle, see [Add or change a traversal move](../player/add_a_traversal_move.md).

---

## The plants

`Content/TheLastTemplate/Meshes/Environments/Vegetation/`, prefixed `SM_Veg_`.

Twenty six pieces: grass tufts and patches, ferns, nettles, sedge, brambles, ivy, hanging vine, a low bush and an elder bush, saplings, an alder, a stump, a snag, a fallen log, a dead branch, leaf litter.

Six of them tile on the same 400 grid so they run along a wall or an edge: `SM_Veg_Bramble_Wall_400`, `SM_Veg_Fence_Growth_400`, `SM_Veg_Fringe_400`, `SM_Veg_Grass_Patch_400`, `SM_Veg_Ivy_Wall_400` and `SM_Veg_Ivy_Wall_Sparse_400`. The rest are single plants you scatter.

**Only the woody pieces block you.** `SM_Veg_Sapling`, `SM_Veg_Sapling_Young`, `SM_Veg_Tree_Alder`, `SM_Veg_Snag`, `SM_Veg_Stump` and `SM_Veg_Log_Fallen` carry collision. Grass, ferns, brambles, ivy and litter carry none, on purpose: you walk through a tuft and you hide inside a bush. If you want a bush to stop the player, add a collision primitive to that mesh.

Four materials dress the whole set, all children of `M_PropsMaster`: `MI_Veg_Bark`, `MI_Veg_Foliage`, `MI_Veg_DryFoliage` and `MI_Veg_Vine`. Wind lives on them, behind a `Use Wind` switch and seven values.

| Field | What it moves |
|---|---|
| `Wind Intensity` | overall strength of the bend |
| `Wind Speed` | how fast the bend cycles |
| `Bend Scale` | how far the whole plant leans |
| `Flutter Amount` | how far a leaf flaps along its own normal |
| `Flutter Speed` | how fast it flaps |
| `Secondary Amount` | a second, slower displacement on top |
| `Secondary Speed` | how fast that second one cycles |

!!! warning "The four vegetation materials must carry the same wind values"
    One plant is split across several of these materials: a leaf uses `MI_Veg_Foliage` while the twig holding it uses `MI_Veg_Bark`. Give one of them a different `Flutter Amount` and the plant comes apart, leaves swinging away from the branch they grow on. Change a wind field on all four, or on none.

How much each part of a plant is allowed to move is painted into the mesh itself, as vertex colours: black where the plant is anchored, rising towards the tips. That is why the stump and the fallen log stand still while a fern tip whips. Your own plant needs the same paint or it will not move like the kit does.

---

## Blocking out before you model

`Content/TheLastTemplate/Materials/Prototyping/` holds a grid material for greyboxing, so you can lay out a level with box brushes and cubes and still read distances off a wall while you build.

`M_PrototypeGrid` is the master, with `MF_ProcGrid` and `MF_ObjectAlignedTexture` as its two material functions. Nine instances ship in `Childs/`.

| Instance | Use |
|---|---|
| `MI_Default` | the general grey |
| `MI_ProtoWall` | vertical surfaces |
| `MI_ProtoGround` | ground |
| `MI_Metal`, `MI_Wood`, `MI_Grass`, `MI_Water`, `MI_Blood` | one per surface, so a greybox already says what it is meant to be |
| `MI_NarrowPassage` | the squeeze gaps the player shuffles through |

The parameters worth touching are `Grid Size` and `Sub Grid Number` for the spacing, `Surface Color`, `Grid Color` and `Sub Grid Color` for the look, and `Top Surface Color`, `Top Grid Color` and `Top Sub Grid Grid Color`, which do the same for faces pointing up so floors read differently from walls. There is also an `Object Aligned` switch, which chooses whether the grid is projected from the world or from the object it is on.

The surface names on those instances are a label for you, not a working surface. A greybox does not throw the right impact until you set `Phys Material Override` on the mesh component. See [How surfaces work](how_surfaces_work.md).

---

## Making the kit look like yours

Every slot on every kit mesh is named after what it is made of, and each slot is filled with the one material instance of the same name. `M_Brick` gets `MI_Prop_Brick`, `M_Concrete` gets `MI_Prop_Concrete`, and so on.

Nine names cover the whole kit: `M_Concrete`, `M_Brick`, `M_Plaster`, `M_Wood`, `M_Metal`, `M_SteelPainted`, `M_Rebar`, `M_ChainLink` and `M_Glass`. The plants add `M_Bark`, `M_Foliage`, `M_DryFoliage` and `M_Vine`.

That means one edit repaints a whole level. Change `MI_Prop_Brick` and every brick in every building follows, because there is no second brick material anywhere.

Eight of these instances carry a world aligned normal map: `MI_Prop_Asphalt`, `MI_Prop_Brick`, `MI_Prop_Concrete`, `MI_Prop_Metal`, `MI_Prop_Plaster`, `MI_Prop_Rebar`, `MI_Prop_SteelPainted` and `MI_Prop_Wood`, plus `MI_Veg_Bark`. The rest are flat colour. For what the master material exposes and how to make your own instance, see [Give your own meshes a material](material_your_own_meshes.md).

---

## What this kit is not

Three things it is worth knowing before you plan a game around it.

- It is **flat colour plus normal maps**. There are no albedo textures on the building pieces. It reads at gameplay distance and it is meant to be replaced, not shipped as a look.
- It is **a shell, not a home**. Two cabinets and a shelf unit are the only furniture. There are no vehicles, no signage beyond one board, no small clutter.
- It is **not tied to any system**. Apart from the three door meshes above, no Blueprint, Data Asset or animation references a kit piece. Deleting the folder breaks nothing else.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
