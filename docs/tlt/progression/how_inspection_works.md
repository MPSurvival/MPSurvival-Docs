# How 3D inspection works

Some objects are not picked up when you press the interact key. They open a screen where the object floats in front of you, lit and isolated, and you turn it in your hands before deciding what to do with it. That is inspection. The books, the notes and the trading cards that ship with the template all use it.

The prompt on those objects is the journal icon, because the base class ships with `Use Icon` on. Its `Interact Text` says `Examine`, and that word replaces the icon if you turn `Use Icon` off.

---

## What the player does

Look at the object, press the interact key, and the inspect screen opens. From there:

| Input | What it does |
|---|---|
| Left mouse button, held | Turn the object |
| Middle mouse button, held | Slide the object inside the frame |
| Mouse wheel | Zoom in and out |
| `IA_InspectAction1` | Run the `Primary` action, `E` by default |
| `IA_InspectAction2` | Run the `Secondary` action, `F` by default |
| `Escape` | Leave the inspect screen |

The mouse and the exit key are plain `Key` fields on `BP_PlayerInspectManager`: `Key Rotate`, `Key Pan`, `Key Zoom In`, `Key Zoom Out` and `Key Back`. They do not go through Enhanced Input, so they do not appear in the rebinding list in the options menu.

The two action buttons do go through Enhanced Input. The component points at `IA_InspectAction1` and `IA_InspectAction2` through its `IA Action 1` and `IA Action 2` fields, and its `Key Act 1` and `Key Act 2` fields are left at `None` on purpose. Change those two keys in `IMC_Default` like any other, see [Change the controls](../start/change_the_controls.md).

---

## The pieces

| Asset | What it is |
|---|---|
| `BP_Interactable_Pickup_InspectableBase` | The actor you place in the level. It is a normal pickup with physics, and it adds one field: `Inspect Data`. |
| `BP_InspectableDataAsset`, the `DA_Inspect_*` assets | What the object is. Name, mesh, start rotation, framing, and the list of actions. |
| `BP_PlayerInspectManager` | A component on the player. It opens and closes the screen, reads the mouse, and holds the feel settings. |
| `BP_InspectStage` | An actor that holds the inspected mesh, three lights and a scene capture. |
| `BP_InspectWidget` | The screen itself, with the object, its name and the two buttons. |

The Data Assets live in `Content/TheLastTemplate/Blueprints/DataAssets/Inspect/Childs/`. `BP_InspectStage` and the action classes are in `Content/TheLastTemplate/Blueprints/Inspect/`. The pickup is in `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/Inspect/`, the component in `Content/TheLastTemplate/Blueprints/ActorComponents/`, and the two widgets in `Content/TheLastTemplate/Blueprints/Widgets/Inspect/`.

Only the three books have their own child class of the pickup. The notes and the cards are placed with `BP_Interactable_Pickup_InspectableBase` itself, with `Inspect Data` set on the placed actor. Both ways work, so make a child class only when you want the same object in several levels.

---

## The stage is filmed, not the level

`BP_InspectStage` carries its own `Item Mesh`, its own `Key Light`, `Fill Light` and `Rim Light`, and a `Scene Capture` pointed at them. The capture writes into the render target `Content/TheLastTemplate/Textures/Widgets/Inspect/RT_Inspect`, which is `1600` by `900`, and the material `M_UI_InspectRT` draws that render target on screen.

Nothing of the room the player is standing in reaches that image. An object looks the same at night in a basement as it does at noon outdoors, which is the point.

The consequence is that an inspectable object has **two meshes**, and they are two different fields:

- `Static Mesh` on the pickup actor, the one lying in the level
- `Mesh` on the `DA_Inspect_*` Data Asset, the one shown on the stage

They are normally the same asset, but nothing forces it. If the world shows a grey cube, `Static Mesh` was never set. If the inspect screen is empty, `Mesh` was never set.

---

## Framing

Two fields on the Data Asset decide how the object presents itself.

| Field | What it does | Shipped values |
|---|---|---|
| `Inspect Rotation` | The rotation the object opens at, before the player turns it | `0 / 90 / 90` on all eight shipped objects |
| `Frame Fill` | How much of the frame the object takes up. Raise it and the object comes closer | `0.75` on the three books, `0.82` on the notes and the cards |

The camera itself is on the stage: `Capture FOV` is `32` and `Capture Aspect` is `1.7777778`.

---

## The two action slots

Every `DA_Inspect_*` has an `Actions` array of `S_InspectActionEntry`. Each entry is one button.

| Field | What it does |
|---|---|
| `Action Class` | The `BP_InspectActionBase` child that runs when the button is pressed |
| `Slot` | `Primary` or `Secondary`, which is the `E` button or the `F` button |
| `Label` | The word written on the button, for example `Read` or `Take` |
| `Text Param`, `Text Lines`, `Float Param`, `Int Param`, `Class Param` | A parameter bag. Each action class reads only the ones it needs and ignores the rest |
| `Closes Inspect` | Whether running the action leaves the inspect screen |

There are only the two slots, so a screen shows at most two buttons. A trading card ships with `Read` on `Primary` and `Take` on `Secondary`. `DA_Inspect_Note_Sorry` ships with `Read` alone.

These action classes ship, under `Content/TheLastTemplate/Blueprints/Inspect/Actions/Childs/`:

| Class | What it does | What it needs |
|---|---|---|
| `BP_InspectAction_Read` | Shows a page of text over the object | `Text Param` as the title, `Text Lines` as the body, on the entry |
| `BP_InspectAction_TakeItem` | Takes the object as an inventory item | `Class Param` as the interactable to hand over, on the entry |
| `BP_InspectAction_TakeCollectible` | Takes the object as a collectible | Nothing on the entry |
| `BP_InspectAction_LearnRecipe` | Learns a crafting recipe | A child class with `Recipe to Learn` set |
| `BP_InspectAction_UnlockSkill` | Unlocks a player upgrade | A child class with `Skill to Unlock` and `Unlocked Levels` set |

The last two ship with their target empty, so each object that uses them points at a child class instead. `BP_InspectAction_LearnRecipe_Molotov` is a child with `Recipe to Learn` set to `DA_Recipe_Molotov`, and `BP_InspectAction_UnlockSkill_HealingPower` is a child with `Skill to Unlock` set to `DA_Skill_HealingPower` and `Unlocked Levels` set to `3`. Duplicate a child, change its target, and you have a new book.

`BP_InspectActionBase` also carries `Hide When Blocked`. When an action is not available, that boolean decides whether its button disappears or stays on screen. It is `true` on every action that ships.

---

## Reading stays on the stage

Every shipped `Read` row sets `Closes Inspect` to `false`. Pressing the button opens `BP_InspectReadWidget` over the object instead of ending the inspection, so the player can read the note and then still press the other button. The shipped `Take` and `Learn` rows set `Closes Inspect` to `true`: once the object has given what it had, there is nothing left to look at.

---

## Feel settings

All of these are on `BP_PlayerInspectManager`, on the player.

| Field | Shipped value | Note |
|---|---|---|
| `Rotate Speed` | `0.9` | How fast the object follows the mouse |
| `Zoom Speed` | `0.14` | How much one wheel step moves the zoom |
| `Pan Speed` | `0.0035` | Much smaller than the other two. Panning is a nudge, not a move |
| `Zoom Min` / `Zoom Max` | `0` / `1` | Zoom is an amount between the two, not a distance |
| `Zoom Start` | `0` | The object opens at `Zoom Min` |
| `Invert Rotate X` / `Invert Rotate Y` | `false` | One axis each, so you can invert only one |

---

## Collectibles reuse the same thing

A collectible is a `BP_CollectibleDataAsset` with three fields: `Display Name`, `Category` and `Inspect Data`. It does not describe a mesh or a screen of its own, it points at a `DA_Inspect_*`. So a trading card is one inspectable object plus one line saying which shelf it belongs on, and the card is claimed by putting `BP_InspectAction_TakeCollectible` in the `Secondary` slot.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
