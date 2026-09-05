# The controls

Input goes through **Enhanced Input**. There is one mapping context, `Content/GrandOpening/Inputs/IMC_Gameplay`, and the actions live next to it in `Inputs/Inputs/`.

The keys live in `IMC_Gameplay`. Changing one means opening that asset and dragging the mapping onto another key, which takes about five seconds.

---

## On foot

| Action | Key | `IA_` asset |
|---|---|---|
| Move | `W` `A` `S` `D` | `IA_Move` |
| Look | mouse | `IA_Look` |
| Sprint | `Left Shift` | `IA_Sprint` |
| Jump | `Space` | `IA_Jump` |
| Crouch | `Left Ctrl` (hold) | `IA_Crouch` |
| Interact | `E` | `IA_Interact` |
| Step back from a screen | `Tab`, or `E` again | `IA_ExitFocus` |
| Pause | `P` | `IA_Pause` |

Sprint has no stamina bar behind it. It swaps `MaxWalkSpeed` from 240 to 420, and the view reacts on its own: the head bob speeds up, widens, and the field of view opens. Crouch is the engine's own `Crouch()`, with the eye height eased down by the view component rather than snapped.

---

## Carrying something

| Action | Key |
|---|---|
| Pick up | `E` |
| Put it down where you are looking | `G` |
| Throw it | `R` |
| Open or close the box | `F` |
| Put one unit on the shelf you are looking at | `Left Mouse` |
| Take one unit back off the shelf | `Right Mouse` |

The keys you can use right now are drawn in a small panel next to whatever you are holding, so you do not have to remember this table in game. The panel is filled by the player first (*Put down*, *Throw*) and then by the object, which adds its own lines. A box adds *Open*, *Stock* and *Take back*, and *Store* appears only when you are looking at a rack that still has a free slot.

Each line reads its letter from the `InputAction` itself. Remap the key and the panel follows.

---

## Build mode

| Action | Key |
|---|---|
| Open the build menu | `B` |
| Place the armed structure | `Left Mouse` |
| Rotate it a quarter turn | `R` |
| Leave build mode | `B` |
| Pick a placed structure back up (edit mode) | `X` |

Clicking a tile in the menu arms that structure and closes the menu, so you go straight back to aiming. Pressing `B` again drops out of build mode entirely.

Edit mode is entered from the `EDIT` button at the bottom right of the build menu. See [How placement works](../build/how_placement_works.md).

---

## In front of a screen

The office computer, the checkout display and the card terminal all work the same way, because they all inherit from `BP_ScreenBase`.

`E` leans you in. The camera moves to the screen's `ReadPose`, the crosshair and the interaction prompt hide, whatever you were carrying is put down, and the mouse cursor appears. From there the screen is a normal mouse surface: hover, click, scroll and drag all work.

`Tab` or `E` leans you back out.

Movement and look are frozen while you are reading.

---

## Changing a key

1. Open `Content/GrandOpening/Inputs/IMC_Gameplay`.
2. Find the row for the action.
3. Click the key field and press the key you want.

Do not delete a row and add a new one if the mapping carries triggers or modifiers, since those live on the row rather than on the action. `IA_Move` in particular carries the four directional modifiers that turn one key into one axis.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
