# Make your own door, switch or lever

This page shows how to build four things of your own. A door that a button or a lever opens. Any other actor that a button or a lever switches. An emitter, such as a pressure plate. And an object the player drags by hand, such as a drawer, a valve or a hatch. If you have not read [How interaction works](how_interaction_works.md), start there.

- Doors: `Content/MPFriendslop/Blueprints/Environments/Doors/`
- Buttons and levers: `Content/MPFriendslop/Blueprints/Environments/Fixtures/`
- The link and the axis: `Content/MPFriendslop/Blueprints/ActorComponents/`
- The parent of hand-dragged objects, `BP_GrabAxisBase`: `Content/MPFriendslop/Blueprints/Environments/`

| Start from | Take this one when | What it brings |
|---|---|---|
| `BP_DoorBase` | A button or a lever opens it | The open or closed state, the timing, the link |
| `BPI_Activatable` | It is not a door, but a button or a lever switches it | Two events to fill in |
| `BP_ActivationLinkComponent` | You make your own emitter: a pressure plate, a zone, a timer | A list of targets to switch |
| `BP_GrabAxisBase` | The player drags it by hand along one axis | Everything, with zero graph |

---

## A door of your own

`BP_DoorBase` already knows whether the door is open and how far along it is. Your child only decides how its meshes move. `BP_SlidingDoor_Double` slides two leaves, and `BP_ExtractionTrap` swings two leaves down by `Open Angle`: open either one to copy from.

1. Right click `BP_DoorBase`, then **Create Child Blueprint Class**. Save it in `Doors/Childs/`.
2. Add your meshes. Keep the fixed frame and the moving parts as separate components, and put the pivot of a swinging part on its hinge.
3. Add the variable your movement needs, for example a travel in cm or an angle in degrees, in the category `Settings|Door` and **Instance Editable**.
4. Override `ApplyDoorAlpha`. Its `Alpha` input goes from `0` (closed) to `1` (open), already eased. Lerp your part from its closed pose to its open pose with it, and set its relative location or rotation.
5. Place it and link a button or a lever to it, as in [Link a button or a lever to a door](doors_buttons_and_levers.md).

That is the whole job. You do not add a Timeline or a Tick: the base moves `Alpha`, and stops ticking once the door has arrived.

| Field | What it does | Shipped default |
|---|---|---|
| `Is Open` | The door state. Tick it on a placed door to start it open | off |
| `Move Duration` | Seconds for a full open or close. `0` is instant | `0.8` |
| `Open Travel` | On `BP_SlidingDoor_Double` only: how far each leaf slides, in cm | `114` |

For a sound or an effect when the door moves, bind `OnDoorStateChanged` on your child.

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnDoorStateChanged` | `Is Open` changed. Gives `Is Open` | every player |

---

## Let a button or a lever switch your own actor

A button or a lever does not know what it switches. It sends one of two messages from `BPI_Activatable` to every actor in its `Targets`:

| Message | Sent by | What your actor does |
|---|---|---|
| `Toggle` | `BP_InteractButton` | Flips its state |
| `SetActivated` | `BP_LeverBase` | Takes the `Activated` value it receives |

1. Open your actor, then **Class Settings**, and add `BPI_Activatable` to **Implemented Interfaces**.
2. Keep the state in a variable set to **RepNotify**, and apply the visual in its `OnRep` function.
3. Implement `Toggle` and `SetActivated`: they only write that variable.
4. In **Class Defaults**, tick `Replicates`.

The lamps are a good first case, because they do not implement `BPI_Activatable` when they ship. Make a child of `BP_LightBase`, add the interface, and in its two events call `SetLightActive` on its `BP_LightComponent`: `Toggle` passes the opposite of `Is Light Active`, `SetActivated` passes `Activated`. The component already keeps the state in a replicated variable, so the lamp needs no variable of its own. More on lamps in [Place, switch and make lamps](../world/lamps_and_lights.md).

---

## Your own emitter: a pressure plate or a zone

Anything can switch targets: add a `BP_ActivationLinkComponent` to it and call it. The template ships no pressure plate, so here is the recipe.

1. Make an actor with a `Box Collision` and a `BP_ActivationLinkComponent`. Tick `Replicates`.
2. On the box's `On Component Begin Overlap`, call `SetTargetsActivated` on the link with `Activated` true.
3. On `On Component End Overlap`, call it with `Activated` false. If several players can stand on it, first check that nothing is left on the box.
4. Place it, and pick its targets in `Targets` with the eyedropper.

| On `BP_ActivationLinkComponent` | What it does |
|---|---|
| `Targets` | The actors it switches, picked in the level |
| `ToggleTargets` | Sends `Toggle` to every target |
| `SetTargetsActivated` | Sends `SetActivated` to every target |

Both functions only act on the server. On a player's machine they do nothing. An overlap fires on every machine, so you can call them from it without checking authority yourself.

---

## An object dragged by hand

A drawer, a valve or a hatch is a child of `BP_GrabAxisBase` with a `BP_GrabAxisComponent`. The player grabs it with the left mouse button and moves it along one axis. `BP_SlidingDoor_Single` is built this way, with no variable and no graph of its own.

1. Right click `BP_GrabAxisBase`, then **Create Child Blueprint Class**.
2. Add the fixed mesh.
3. Add a `BP_GrabAxisComponent` under it, placed exactly on the hinge, or at the start of the rail.
4. Attach the moving mesh under the `BP_GrabAxisComponent`. Leave it in its closed pose: `Min Value` and `Max Value` are measured from that pose.
5. Fill `Settings|Axis` on the `BP_GrabAxisComponent`.
6. Optional: add a `BP_OutlineComponent` and list your mesh names in its `Mesh Names`.

| Field | What it does | Default | Lever | Single door |
|---|---|---|---|---|
| `Axis Type` | `Angular` for a hinge, `Linear` for a rail | `Angular` | `Angular` | `Linear` |
| `Axis` | The axis it turns around or slides along, in its parent's space | `(0,0,1)` | `(-1,0,0)` | `(1,0,0)` |
| `Min Value` | Degrees or cm at the closed end | `0` | `-45` | `0` |
| `Max Value` | Degrees or cm at the open end | `90` | `115` | `115` |
| `Snap Positions` | `0` is free. `2` snaps to closed or open, `3` adds a middle notch | `0` | `2` | `0` |
| `Snap Speed` | How fast it settles into the nearest notch after release | `4` | `4` | `2` |
| `Drag Sensitivity` | How much it moves for a given gesture. Lower feels heavier | `1` | `0.25` | `0.7` |
| `Follow Speed` | How closely it follows the hand. Lower lags more | `8` | `6` | `5` |
| `Release Damping` | Without notches, how fast it slows down after you let go | `4` | `0` | `4` |
| `Smoothing Speed` | How smoothly the pose is drawn on the other players' screens | `12` | `12` | `12` |
| `Drive Speed` | When it moves by itself, for example a door that an enemy bumps, how many full strokes per second | `1` | `1` | `1` |

To have enemies open it too, see [Doors the Warden can open](../ai/doors_the_warden_can_open.md).

To react to the position, bind `OnAxisAlphaChanged`. `GetAxisValue` gives the current angle or distance.

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnAxisAlphaChanged` | The position changed. Gives `Alpha`, from `0` to `1` | every player |

To make a handle that switches other things, duplicate `BP_LeverBase` and swap its meshes. It sends `SetTargetsActivated` when its `Alpha` crosses `Activation Alpha` (`0.5`).

---

## Mistakes that cost time

- **A target that does not implement `BPI_Activatable` does nothing.** There is no cast and no error message.
- **`SetTargetsActivated` called only on a player's machine does nothing.** Call it from a path that runs on the server.
- **One axis per actor.** `BP_GrabAxisBase` uses the first `BP_GrabAxisComponent` it finds.
- **A hand-dragged door cannot also be opened by a button.** `BP_SlidingDoor_Single` is not a `BP_DoorBase`, and one position cannot have two owners.
- **Moving parts do not push players.** Doors and axis parts are moved without sweep.

!!! warning
    An actor without `Replicates` ticked still works on the host, and nothing happens on the other players' screens. There is no error message.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
