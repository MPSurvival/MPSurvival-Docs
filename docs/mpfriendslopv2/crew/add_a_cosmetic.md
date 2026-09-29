# Add a hat, an accessory or a pattern

A cosmetic is one Data Asset, plus one line in the catalogue. By the end of this page your piece has a tile in the **Customization** screen, a price in credits, and shows on the robot in the menu and on every player in a match.

If you have not read [How cosmetics work](how_cosmetics_work.md), start there.

- The Data Assets: `Content/MPFriendslop/Blueprints/DataAssets/Cosmetic/Childs/`
- The meshes: `Content/MPFriendslop/Meshes/Characters/Cosmetics/`
- The icons: `Content/MPFriendslop/Textures/Widgets/Cosmetics/`
- The catalogue: `Content/MPFriendslop/Blueprints/Misc/BP_FriendslopGameInstance`

---

## Step 1, the mesh

1. Import your static mesh into `Meshes/Characters/Cosmetics/`.
2. Remove its collision. No shipped cosmetic has any collision, so a hat never blocks its own player's traces.
3. Set `Collision Complexity` to `Use Simple Collision As Complex`.
4. If a part of it should take the player's colour, give that part a material slot **named** `M_Robot_Shell` and use `MI_Robot_Shell` on it. The colour and the pattern are applied by slot name, not by slot index, so the slot can sit anywhere in the list.

Parts with a fixed colour keep their own material. The traffic cone does this with `MI_Cosmetic_ConeOrange` next to its shell slot.

---

## Step 2, the Data Asset

1. In `Blueprints/DataAssets/Cosmetic/Childs/`, duplicate `DA_Cosmetic_TopHat` for a hat or `DA_Cosmetic_Backpack` for something on the back.
2. Name it `DA_Cosmetic_<Name>`.
3. Fill in the fields below.

| Field | What it does | Shipped in DA_Cosmetic_TopHat |
|---|---|---|
| `Display Name` | The name on the tile | `Top Hat` |
| `Slot` | The tab it is listed in, and the slot it equips into. A player wears one item per slot | `Hat` |
| `Icon` | The tile picture. Draw it white on a transparent background | `T_Icon_Cosmetic_TopHat` |
| `Price` | The cost in credits. `0` means free | `350` |
| `Locked By Default` | Shows the tile faded, with no price. It cannot be bought | off |
| `Static Mesh` | The rigid piece attached to the character | `SM_Cosmetic_TopHat` |
| `Skeletal Mesh` | A piece that follows the body instead, driven by the character's own animation. None of the shipped cosmetics uses it | empty |
| `Attach Socket` | The socket on the character mesh the piece is attached to | `HatSocket` |

There is no offset field. The socket **is** the offset: to move the piece, move the socket or the mesh pivot. `HatSocket` (on the `head` bone) and `BackSocket` (on `spine_04`) are on the skeleton `SKEL_Mannequin` and on the mesh `SKM_Robot_Courier`.

---

## Step 3, the catalogue

1. Open `BP_FriendslopGameInstance`.
2. In `Settings|Cosmetics`, add an entry to `Cosmetic Catalog`.
3. Pick your Data Asset in the new entry.
4. Save.

!!! warning
    `Cosmetic Catalog` is the only list. A cosmetic that is not in it has no tile, cannot be bought, and the server refuses to equip it. There is no error message.

---

## A pattern

A pattern is a cosmetic with no mesh. It paints a motif over the player's body, on the `M_Robot_Shell` slot only. Six ship: Plain, Bands, Dots, Checker, Grid and Hazard, indices `0` to `5`.

1. Duplicate `DA_Cosmetic_PatternBands`.
2. Set `Slot` to `Pattern` and leave `Static Mesh` and `Attach Socket` empty.
3. Fill in the pattern fields.
4. Add it to `Cosmetic Catalog`, like any cosmetic.

| Field | What it does | Shipped in DA_Cosmetic_PatternBands |
|---|---|---|
| `Pattern Index` | Which of the drawn patterns: `0` Plain, `1` Bands, `2` Dots, `3` Checker, `4` Grid, `5` Hazard | `1` |
| `Pattern Color` | The colour of the motif | set per pattern |
| `Icon Material` | The tile picture as a material instead of a texture | `MI_Widget_PatternBands` |

The same index with another `Pattern Color` is a new pattern already: nothing else to do.

The tile icon is an instance of `M_Widget_Pattern` in `Materials/Widgets/`. For your own, duplicate one of the six `MI_Widget_Pattern*` and set its `PatternIndex`.

A seventh pattern, index `6`, has to be drawn first. The body patterns come from `M_Prop_Master`, group `Robot Pattern`, and a new index needs a new branch in its pattern selection. `M_Widget_Pattern` draws the icon from the same `PatternIndex`, so it needs the same branch.

---

## A new slot

The tabs of the Customization screen are built from `E_CosmeticSlot`, which ships with `Hat`, `Accessory` and `Pattern`.

1. Open `Blueprints/Enumerations/E_CosmeticSlot` and add an entry. Its display name is the tab label.
2. Add a socket for it on your character mesh.
3. Make Data Assets with that `Slot` and that `Attach Socket`, and add them to `Cosmetic Catalog`.

The new tab appears on its own, and a player can now wear one item in it next to the hat and the accessory.

---

## What you get for free

Once the Data Asset is in `Cosmetic Catalog`:

- It has a tile in its tab, with its icon, its name and its price.
- Buying it takes credits and saves it in the player's profile.
- The robot in the main menu wears it while the player looks at it, before buying.
- It is attached to the player in a match, and hidden from the player's own first person view.
- Its `M_Robot_Shell` slot takes the player's colour and pattern.

None of that needs a line of Blueprint.

---

To add a gesture or a face to the emote wheel, see [Add an emote or a face](add_an_emote_or_a_face.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
