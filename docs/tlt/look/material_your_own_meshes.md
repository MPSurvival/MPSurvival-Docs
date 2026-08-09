# Give your own meshes a material

The props, the modular kit, the plants, the weapons and the pickups all run on **one master material**. You never open it. You make a Material Instance from it, set a few fields, and drop the instance on your mesh slot.

- The master: `Content/TheLastTemplate/Materials/Environments/Props/M_PropsMaster`
- The prop and item instances: `Content/TheLastTemplate/Materials/Environments/Props/`
- The plant instances: `Content/TheLastTemplate/Materials/Environments/Vegetation/`

Characters are not on it. They have their own materials under `Materials/Characters/`.

---

## One instance per material slot name

The meshes that ship name their material slots after what the surface is, not after the object. A crate, a door and a pallet all carry a slot called `M_Wood`. A railing and a locker both carry `M_Metal`.

So the slot name is the assignment: slot `M_Wood` gets `MI_Prop_Wood`, slot `M_Brick` gets `MI_Prop_Brick`, slot `M_Foliage` gets `MI_Veg_Foliage`. Filling a freshly imported mesh is a lookup, not a decision.

Name your own meshes' slots the same way and you inherit that. If a mesh needs a look that does not exist yet, add one instance to the library rather than a one-off material on that mesh.

---

## The fields you set

Open any instance and you get four groups in the Details panel.

### `01 - Surface`

| Field | What it does | Master default |
|---|---|---|
| `Base Color` | The flat colour. Also the tint when `Use Texture` is on | set per instance |
| `Use Texture` | Off: flat colour. On: `Base Color Texture` multiplied by `Base Color` | off |
| `Base Color Texture` | Only read when `Use Texture` is on | white |
| `Metallic` | 1 for bare metal, 0 for everything else | 0 |
| `Roughness` | 0 is a mirror, 1 is chalk | 0.6 |
| `Specular` | How strong the highlight is on a non metal | 0.5 |
| `Opacity` | Only does something if you also set the blend mode to translucent | 1 |
| `Subsurface Color` | The colour light takes when it passes through a leaf | set per instance |

### `02 - Emissive`

| Field | What it does | Master default |
|---|---|---|
| `Emissive Color` | The colour the surface gives off | set per instance |
| `Emissive Intensity` | Multiplier on it. Leave at 0 for anything that does not glow | 0 |

### `Normal`

| Field | What it does | Master default |
|---|---|---|
| `Use Normal` | Turns the surface detail on. Off means a perfectly flat surface | off |
| `Normal Texture` | Which tiling map to project | set per instance |
| `Normal Tiling Cm` | The size in world centimetres of one repeat of the map | 200 |
| `Normal Intensity` | 0 flattens the map out, above 1 exaggerates it | 1 |
| `Normal Follows Object` | Off: the projection is fixed in the world. On: it rides the object | off |

### `03 - Wind`

| Field | What it does | Master default |
|---|---|---|
| `Use Wind` | Turns the whole wind chain on | off |
| `Bend Scale` | Overall bend of the plant | 1 |
| `Wind Intensity` | Feeds the bend. Small on purpose | 0.12 |
| `Wind Speed` | Speed of the bend | 0.25 |
| `Wind Direction` | Which way the wind pushes | set per instance |
| `Flutter Amount` | Flap along the leaf normal, in centimetres | 1.2 |
| `Flutter Speed` | Speed of that flap | 2.2 |
| `Secondary Amount` | A second, slower bend, in centimetres | 2 |
| `Secondary Speed` | Speed of that second bend | 0.55 |

Blend mode, two sided and shading model are not parameters. They live in the **Material Property Overrides** section of the instance, and the shipped instances use them: `MI_Prop_Glass` sets `Blend Mode` to `Translucent` and `Two Sided`, `MI_Veg_Foliage` and `MI_Veg_DryFoliage` set `Shading Model` to `Two Sided Foliage` because a leaf is a single sided card.

---

## Flat colour, or a texture

Everything is flat colour by default. That is what makes the kit cheap to dress: a value in `Base Color`, a `Roughness`, a `Specular`, done.

When you do need a texture, tick `Use Texture` and set `Base Color Texture`. That is how the collectible cards and the readable notes work, for example `MI_Prop_Card_Comet` with `T_Card_Comet_BC`.

!!! warning
    With `Use Texture` on, `Base Color` stops being the colour and becomes a **tint**: the texture is multiplied by it. Leave a colour in there and your texture renders darker than the file, with no error anywhere. Set `Base Color` to white unless you actually want to tint it.

---

## Surface detail: the tiling normal maps

Fourteen tiling normal maps ship in `Content/TheLastTemplate/Textures/Environments/Surfaces/`. They are projected on world axes, not on the mesh UVs, so they do not care how a mesh was unwrapped and two walls standing side by side get bricks of the same size.

Each map was drawn at a real physical size. That number, in centimetres, is what goes in `Normal Tiling Cm`.

| Map | What it is | Real size of one tile |
|---|---|---|
| `T_Brick_N` | Brick 190 by 57, mortar joint 10 | 200 cm |
| `T_Plaster_Cracked_N` | Render torn off the brick underneath | 200 cm |
| `T_Concrete_Worn_N` | Form marks, blowholes, spalling, cracks | 200 cm |
| `T_Concrete_Floor_N` | 1 m slabs with sawn joints | 200 cm |
| `T_Asphalt_N` | Aggregate 5 to 20, cracks and crazing | 200 cm |
| `T_Gravel_N` | Loose stones 20 to 70, three sizes | 100 cm |
| `T_Tiles_Ceramic_N` | 300 mm floor tiles, 6 mm joint | 120 cm |
| `T_Wood_Planks_N` | Twelve boards, knots, end joints | 200 cm |
| `T_Wood_Bark_N` | Vertical plates, deep furrows | 100 cm |
| `T_Foliage_Veins_N` | Leaf veining, tiling, not one drawn leaf | 15 cm |
| `T_Roof_Shingles_N` | Shingles 333, 125 exposed, granules | 100 cm |
| `T_Roof_Corrugated_N` | 83 mm wave, overlaps, screws | 100 cm |
| `T_Metal_Rusted_N` | Pitting and scales lifting off | 100 cm |
| `T_Metal_Tread_N` | 3 mm checker plate | 50 cm |

Set `Normal Tiling Cm` to that number and the detail is at true scale. A smaller number makes it finer and repeats more often, which is sometimes what you want on a small prop. The instances that already carry a map are `MI_Prop_Asphalt`, `Brick`, `Concrete`, `Metal`, `Plaster`, `Rebar`, `SteelPainted`, `Wood` and `MI_Veg_Bark`, and not all of them are set to the true size.

Every map has an `_ORH` companion next to it, with ambient occlusion in red, roughness in green and height in blue. `M_PropsMaster` does not read it. It is there for you if you want to build a richer material of your own.

### `Normal Follows Object`

The projection is fixed in the world by default. That is right for walls, floors and anything that never moves: the texture runs continuously across the joint between two modular pieces.

It is wrong for anything that moves. A door swinging on its hinge, a crate you pick up, a backpack on a character: the object slides through a texture that stays where it was. Tick `Normal Follows Object` on those instances and the projection travels with the mesh instead.

The cost is real and you should know it before you tick it: the projection then works in object space, so a static mesh that you scaled in the level gets its detail scaled with it, and two abutting pieces no longer line up. Pick one behaviour per instance, and make a second instance if one mesh needs the other.

---

## Make your own instance

1. Right click `M_PropsMaster`, then **Create Material Instance**. Name it after the slot: `MI_Prop_Ceramic`, `MI_Veg_Moss`. Put prop and item looks in `Materials/Environments/Props/`, plants in `Materials/Environments/Vegetation/`.
2. Tick `Base Color` and pick the colour. Tick `Roughness` and `Specular` and set them. Set `Metallic` to 1 only if the surface is bare metal.
3. If it needs a texture, tick `Use Texture`, set `Base Color Texture`, and set `Base Color` to white.
4. If it needs relief, tick `Use Normal`, pick a `Normal Texture`, and set `Normal Tiling Cm` to that map's real size. Tick `Normal Follows Object` if the mesh moves.
5. Assign the instance to the mesh's material slot.

For a transparent surface, add `Blend Mode` in **Material Property Overrides** and set it to `Translucent`, then use `Opacity`. `MI_Prop_Glass` is the worked example.

---

## Plants and their wind

Wind is off unless the instance ticks `Use Wind`. When it is on, the master bends the mesh with `Bend Scale`, `Wind Intensity` and `Wind Speed`, then adds two more movements on top: `Flutter Amount` along the leaf normal, and `Secondary Amount` as a slower second bend. Those last two are added in centimetres and carry most of what you actually see.

How much a given vertex moves is a **vertex colour** painted into the mesh, black at the root and white at the tips. The material only sets the gain.

Two rules that save a lot of time:

- **Every instance used by one plant must carry the same wind values.** A plant is several slots, bark plus foliage, or vine plus foliage. If two of them run at different speeds or gains the plant comes apart while it moves. The four shipped instances all carry `Bend Scale` 0.9, `Wind Intensity` 0.1, `Wind Speed` 0.25, `Flutter Amount` 0.3, `Flutter Speed` 2.2, `Secondary Amount` 0.9, `Secondary Speed` 0.5. Retune all of them together or none.
- **Stiffness is the mesh's job, not the material's.** Dead wood is black in the vertex colours, so it stays still with the same values a leaf uses. Do not try to express "wood is stiffer" by lowering the gains, or you flatten the leaves too.

!!! warning
    If you import your own plant FBX and it has no vertex colours, the material reads white everywhere and the plant waves from the root, including the trunk. Import vegetation by dragging the FBX into the Content Browser so the import dialog appears, and set **Vertex Color Import Option** to `Replace`.

## The other materials in the pack

Not everything is an instance of the props master.

| What | Where |
|---|---|
| Water | `Materials/Environments/Water/M_Ocean` and `MI_Ocean` |
| Post process | `Materials/Effects/PostProcess/M_TLTPostProcess` and `MI_TLTPostProcess` |
| Block out grids | `Materials/Prototyping/M_PrototypeGrid` and the instances in `Prototyping/Childs/` |
| Menu background | `Materials/Environments/Menu/` |
| Material parameter collections | `Materials/Collections/` |

Impact decals and the Niagara sheets have their own masters too.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
