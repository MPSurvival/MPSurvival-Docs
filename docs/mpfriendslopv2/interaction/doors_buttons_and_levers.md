# Link a button or a lever to a door

By the end of this page you have a door in your level that opens and closes from a button, a lever, or both. No graph to open: you place a few actors and pick the door with the eyedropper.

If you have not read [How interaction works](how_interaction_works.md), here is the short version. The button or lever holds a list of `Targets`. Every target gets a message when it fires.

- The doors: `Content/MPFriendslop/Blueprints/Environments/Doors/` (`BP_SlidingDoor_Double` is in its `Childs/` subfolder)
- The button and the lever: `Content/MPFriendslop/Blueprints/Environments/Fixtures/`

---

## The pieces

| Actor | What it is |
|---|---|
| `BP_SlidingDoor_Double` | A double sliding door. Its two leaves slide into the wall. It opens only when something tells it to |
| `BP_InteractButton` | A button you use with `E`. Each use flips its targets: open becomes closed, closed becomes open |
| `BP_LeverBase` | A lever you drag with the grab button (`Left Mouse Button`). Pull it past its threshold and it is on, push it back and it is off |

`BP_SlidingDoor_Single` is a different kind of door: players drag it open by hand. It has no link, and a button cannot open it. Pick `BP_SlidingDoor_Double` for anything a button or lever drives.

---

## Place the door

The leaves slide sideways into the wall, so the wall around the door needs room for them.

1. Place `SM_Mod_Wall_Doorway_400` as the wall that holds the door.
2. Place one `SM_Mod_Wall_DoorPocket_200` on each side of it. In the doorway wall's space: the right one at `(400, 0, 0)`, the left one at `(0, 0, 0)` rotated `180` on Z. The leaves slide into these.
3. Drag `BP_SlidingDoor_Double` into the level and put it at `(200, 0, 0)` in the doorway wall's space. Its pivot is the centre of the doorway at floor level.
4. If the door must start open, tick `Is Open` on the placed door.

Use the pockets, not a plain wall: a plain wall beside the door sits right in the leaves' path.

---

## Link a button

1. Drag `BP_InteractButton` into the level and put it on the wall next to the door, out of the leaves' path.
2. Select it, then select its `BP_ActivationLinkComponent` in the Details panel.
3. Add an entry to `Targets` and use the eyedropper to pick the door.
4. Set `Prompt`, `Type` and `Duration` on the button.

A button has no state of its own and no press animation. What the player sees is the door moving.

| Field | Where | What it does | Default |
|---|---|---|---|
| `Targets` | the button's `BP_ActivationLinkComponent` | The actors this button flips. Add as many as you want | empty |
| `Prompt` | the button | The line under the ring. The shipped map uses `Toggle Door` | `Press Button` |
| `Type` | the button | `Simple`, `Hold` or `Spam`. How the player uses it | `Simple` |
| `Duration` | the button | Seconds to hold the key when `Type` is `Hold` | `0` |

---

## Link a lever

1. Drag `BP_LeverBase` into the level and put it on the wall. Its base sits flat against the wall.
2. Select its `BP_ActivationLinkComponent`.
3. Add an entry to `Targets` and pick the door with the eyedropper.

The lever has two notches. Once it passes `Activation Alpha`, it tells its targets "on", and when it comes back below, "off". A door takes on as open and off as closed.

| Field | Where | What it does | Default |
|---|---|---|---|
| `Targets` | the lever's `BP_ActivationLinkComponent` | The actors this lever switches | empty |
| `Activation Alpha` | `BP_LeverBase` | How far along its travel, from 0 to 1, the lever counts as on | `0.5` |

How the lever feels in the hand (its travel, its notches, how heavy it is to drag) is covered in [Make your own door, switch or lever](make_your_own_door_or_lever.md).

---

## The door's fields

| Field | Where | What it does | Default |
|---|---|---|---|
| `Is Open` | `BP_DoorBase` | The door's state. Tick it on a placed door to start open | off |
| `Move Duration` | `BP_DoorBase` | Seconds for a full open or close. `0` snaps it | `0.8` |
| `Open Travel` | `BP_SlidingDoor_Double` | How far each leaf slides, in cm | `114` |

A door that is not moving does not tick, so a level full of closed doors costs nothing.

---

## Things to know

- **Put a control on each side of the door.** The shipped map does this. A door closed from one side with its only button on the other side traps the players.
- **One control can drive several targets.** Add more entries to `Targets`: one button can open two doors, or a door and the extraction hatch. The hatch is linked the same way, see [Place an extraction point](../loot/place_an_extraction_point.md).
- **A button and a lever on the same door both work,** but they speak differently. The button flips the door. The lever says on or off. So after someone presses the button, the lever's position no longer matches the door.
- **The leaves do not push players.** They move straight to their position without checking what is in the way.

!!! warning
    A target that cannot be switched does nothing, with no error. If you pick `BP_SlidingDoor_Single`, a wall or any actor that does not implement `BPI_Activatable`, the button still shows its prompt and nothing happens.

---

## The dispatchers

| Dispatcher | Where | When it fires | Fires on |
|---|---|---|---|
| `OnDoorStateChanged` | `BP_DoorBase` | The door starts to open or close. Gives `Is Open` | every player |
| `OnAxisAlphaChanged` | the lever's `BP_GrabAxisComponent` | The lever moved. Gives `Alpha`, from 0 to 1 | every player |

---

To make a button switch your own actor, or to build your own door, see [Make your own door, switch or lever](make_your_own_door_or_lever.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
