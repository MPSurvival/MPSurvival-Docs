# Add a box size

Boxes are not scaled. Each size is a real mesh with real cardboard thickness, its own flaps and its own collision hull, described by one Data Asset.

Three ship with the template: `DA_Box_Small`, `DA_Box_Medium`, `DA_Box_Large`.

---

## Why not just scale one

Scaling a box multiplies everything under it. The products inside grow with the carton, the mass and inertia go wrong, and the cardboard thickness doubles, which breaks the clearance the flaps need to swing without intersecting the body.

So: sizes, not scale.

---

## The recipe

1. Model the box at its real size, with body, front flap and back flap as separate pieces, and export it.
2. Import the meshes into `Meshes/Props/Boxes/`.
3. Create a `DA_Box_<Size>` in `Blueprints/DataAssets/Boxes/Childs/`.
4. Fill it in, using the measurements from the model rather than from the eye.
5. Duplicate `BP_DeliveryBox_Medium` into `Environments/Boxes/Childs/`, rename it, and set its `Size` to your new asset. That is the only thing the child does.
6. Add your `DA_Box_*` to `BoxSizes` on `BP_DeliveryManager`, in the Details panel of the game state, so deliveries can pick it.

---

## The fields

| Field | What it does |
|---|---|
| `BodyMesh` | The carton |
| `FlapMesh` | The back flap |
| `FlapLabelMesh` | The front flap, the one that carries the docket |
| `FlapHingeYCm` | Where the flaps hinge, across the box |
| `FlapHingeZCm` | Where the flaps hinge, up the box |
| `InteriorMinCm` | The inside corner, in the box's local space |
| `InteriorSizeCm` | The usable interior volume |

`InteriorMinCm` and `InteriorSizeCm` are the two that matter for gameplay. Everything about how much fits inside comes from them:

```
capacity =  floor( InteriorSizeCm.X / product FootprintCm.X )
          × floor( InteriorSizeCm.Y / product FootprintCm.Y )
          × min( product StackMax , floor( InteriorSizeCm.Z / product StackHeightCm ) )
```

Measure the inside of the mesh, not the outside. A box measured on its outer face holds units that stick through the cardboard.

---

## The docket position is set by hand

The white docket and the product icon sit on the front flap. Their position is a component transform on the child Blueprint, not a field on the Data Asset. That is deliberate: where the label goes is a consequence of how the mesh is modelled, not a piece of content someone fills in.

If the icon comes out on the wrong side after an import, it is the FBX export flipping Y. X and Z survive the trip, Y comes out mirrored.

---

## Two traps worth knowing

**`InitialUnits` cannot exceed the real capacity.** Set it higher and the extra units are refused with no warning. If a delivery of one case comes up short, either `UnitsPerCase` on the product is too high or the largest box is too small.

**Adding a component to a Blueprint does not update actors already placed in a level.** A box already sitting in your map keeps the engine defaults for anything you add afterwards, and no amount of property writing brings it back in line. Delete the placed actor and place a fresh one. The same is true of new variables on a parent class: they come out empty on children that were already compiled.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
