# Add a box size

Boxes are not scaled, they are modelled. Each size is a real mesh with real cardboard thickness, its own flaps and its own collision hull, described by one Data Asset.

Three ship with the template: `DA_Box_Small`, `DA_Box_Medium`, `DA_Box_Large`.

---

## The recipe

1. Model the box at its real size, with body, front flap and back flap as separate pieces, and export it.
2. Import the meshes into `Meshes/Props/Boxes/`.
3. Create a `DA_Box_<Size>` in `Blueprints/DataAssets/Boxes/Childs/`.
4. Fill it in with the measurements from the model.
5. Duplicate `BP_DeliveryBox_Medium` into `Environments/Boxes/Childs/`, rename it, and set its `Size` to your new asset. That is all the child does.
6. Add your `DA_Box_*` to `BoxSizes` on `BP_DeliveryManager`, in the game state's Details panel, so deliveries can pick it.

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

`InteriorSizeCm` is the one that decides how many units a box holds, against the product's footprint, stack height and `StackMax`.

**Measure the inside of the mesh, not the outside.** A box measured on its outer face holds units that stick through the cardboard.

---

## The docket

The white docket and the product icon sit on the front flap, positioned as components on the child Blueprint rather than as fields on the Data Asset, because where the label goes depends on how the mesh was modelled.

If the icon comes out on the wrong side after an import, it is the FBX export flipping Y. X and Z survive the trip, Y comes out mirrored.

---

## Two traps

**`InitialUnits` cannot exceed the real capacity.** Set it higher and the extra units are dropped with no warning. If a delivery of one case comes up short, `UnitsPerCase` on the product is too high or the largest box is too small.

**A box already placed in a level does not gain components you add afterwards.** Delete the placed actor and place a fresh one. Same for new variables on a parent class: they come out empty on children that were already compiled.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
