# Add or change a traversal move

The player vaults, hurdles, mantles and climbs with a single key, `IA_Vault`, on `Space` by default. The obstacle decides which of those you get. A Data Asset decides which animation plays.

Six moves ship, one Data Asset each:

`Content/TheLastTemplate/Blueprints/DataAssets/Traversal/Childs/`

They are listed on the player. Open `Content/TheLastTemplate/Blueprints/PlayerCharacter/BP_PlayerCharacter` and look for the array `Traversal Actions`. An asset that is not in that array is never used.

---

## The six moves that ship

| Asset | `Action Type` | `Montage` |
|---|---|---|
| `DA_Traversal_Vault_Stand_01` | `Vault` | `AM_Character_Traversal_Vault_01` |
| `DA_Traversal_Hurdle_Stand_02` | `Hurdle` | `AM_Character_Traversal_Hurdle_01` |
| `DA_Traversal_Hurdle_Stand_01` | `Hurdle` | `AM_Character_Traversal_Hurdle_02` |
| `DA_Traversal_Mantle_Stand_01` | `Mantle` | `AM_Character_Traversal_Mantle_01` |
| `DA_Traversal_Mantle_Stand_02` | `Mantle` | `AM_Character_Traversal_Mantle_02` |
| `DA_Traversal_Climb_Stand_01` | `Mantle` | `AM_Character_Traversal_Climb_01` |

The bands go with the family. `DA_Traversal_Vault_Stand_01` and `DA_Traversal_Hurdle_Stand_02` cover a depth of 0 to 25 cm, `DA_Traversal_Hurdle_Stand_01` covers 25 to 60. The two mantles cover a height of 0 to 150 cm, `DA_Traversal_Climb_Stand_01` covers 150 to 275. The pair a family does not read is left at `0` on all six.

All six montages live in `Content/TheLastTemplate/Animations/Montages/Characters/Traversal/`.

Note the two hurdle assets: `Hurdle_Stand_01` plays the montage named `Hurdle_02`, and `Hurdle_Stand_02` plays `Hurdle_01`. The names are crossed. Read the `Montage` field, not the asset name.

---

## The fields on a traversal Data Asset

Seven fields, and that is the whole asset.

| Field | What it does |
|---|---|
| `Action Type` | Which family this animation belongs to: `None`, `Hurdle`, `Vault` or `Mantle`. The asset is only ever considered when the obstacle check has already decided on that same type. |
| `Montage` | The montage that plays. |
| `Height Min` / `Height Max` | The band of obstacle heights this asset covers, in centimetres. This is what separates the mantles from each other. |
| `Depth Min` / `Depth Max` | The band of obstacle depths this asset covers, in centimetres. This is what separates the two hurdles from each other. |
| `Speed Min` | An approach speed floor. It is `0` on all six shipped assets, so no move is gated on how fast you were moving. |

## Climb is a tall mantle

`E_TraversalActionType` has four values: `None`, `Hurdle`, `Vault`, `Mantle`. There is no `Climb` entry to pick.

So `DA_Traversal_Climb_Stand_01` declares `Action Type` `Mantle`, and is kept apart from the two ordinary mantles by height alone. They stop at `Height Max` `150`. It starts at `Height Min` `150` and runs to `275`.

If you add a climb of your own, copy that pattern: set `Mantle`, and give it a height band that sits above the mantles.

---

## Add a new move

1. In `Content/TheLastTemplate/Blueprints/DataAssets/Traversal/Childs/`, right click a close relative of what you want, then **Duplicate**. A new mantle starts from `DA_Traversal_Mantle_Stand_01`.
2. Open it and set `Montage` to your montage.
3. Set `Action Type` to the family your animation actually performs.
4. Set the band that matters for that family, and leave the other pair at `0`. Depth for a hurdle, height for a mantle.
5. Open `BP_PlayerCharacter`, find `Traversal Actions`, add an entry and point it at your asset.
6. Save and play.

If you want your move to replace an existing one instead of joining the draw, narrow the band of the old asset so the two do not overlap, or remove the old asset from `Traversal Actions`.

## What your montage has to carry

Every shipped traversal montage carries three kinds of notify. Copy them onto yours.

**Motion warping.** The character sets warp targets from the ledge it measured, and the montage bends itself onto them. Three target names are used, and the spelling has to match exactly.

| Warp target | Meaning | Used by |
|---|---|---|
| `FrontLedge` | The edge you put your hands on | All six montages |
| `BackLedge` | The far edge of the obstacle | All six montages |
| `BackFloor` | The ground behind the obstacle | The two hurdle montages only |

A montage that never mentions `FrontLedge` will play at the animation's own scale and miss the obstacle.

**`ANS_TraversalIK`.** This notify state turns the hand IK on in the animation Blueprint for as long as it runs, so the hand lands on the edge that was measured rather than where the animator put it. It has two checkboxes, `Enable Left` and `Enable Right`. `AM_Character_Traversal_Hurdle_02` enables only the right hand, the other five enable both.

That correction is not free. It only holds while the notify runs, and it drags the hand from wherever your animation had it to the real edge. Keep the notify over the frames where the hand is meant to be planted, and author the animation on an obstacle close to the middle of your band, or the arm will visibly stretch to reach.

**`ANS_BlendOut`.** This ends the montage early. It has `Blend Out Condition`, which is one of `Force`, `MovementInput` or `Falling`, and `Blend Out Time`. It is what lets you break out of the tail of a vault by pushing the stick, instead of waiting for the animation to finish.

For getting your own animations onto the character in the first place, see [Use your own animations](use_your_own_animations.md).

---

## Make your own geometry traversable

A static mesh dropped in the level is never traversable, however low it is. The forward trace has to hit a `BP_TraversableBlock`.

That Blueprint is at `Content/TheLastTemplate/Blueprints/Environments/Traversal/BP_TraversableBlock`, with one child shipped as an example, `BP_Traversable_Container`, a twenty foot shipping container with four ledges drawn on it.

| Field | What it does |
|---|---|
| `Ledge_1` to `Ledge_4` | Spline components. Each spline is one edge the character may grab. Draw them along the top edges of your mesh. |
| `StaticMesh` | The component holding the mesh the block draws and collides with. |
| `Min Ledge Width` | Shipped `60`. A ledge narrower than this, in centimetres, is not used. |
| `Block Visible` | Shipped `true`. Shows or hides the block's own mesh. |

To make your own:

1. Right click `BP_TraversableBlock`, then **Create Child Blueprint Class**.
2. Set the mesh on `StaticMesh` to your own.
3. Move each `Ledge_` spline onto an edge you want the player to be able to grab, and shape it to follow that edge.
4. Delete or shrink the splines you do not need.
5. Drop it in the level and walk into it.

Only the edges you drew a spline on are traversable. A crate with one spline on its front edge can be climbed from the front and nowhere else, which is usually what you want in a level you are blocking out.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
