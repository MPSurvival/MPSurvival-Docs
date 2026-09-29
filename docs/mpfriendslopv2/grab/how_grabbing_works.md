# How grabbing works, and tuning the weight

Loot, levers and hand-dragged doors are grabbed, not used with `E`. Hold `Left Mouse` on the object, move, and let go to drop it. Like interaction, it is one interface on the object and one component on the player.

- The interface: `Content/MPFriendslop/Blueprints/Interfaces/BPI_Grabbable`
- The component: `Content/MPFriendslop/Blueprints/ActorComponents/BP_GrabComponent`

The player never knows what it holds. The object says whether it can be grabbed, how it moves and how it should sit in the hand. A new kind of grabbable never needs an edit on the player.

This page is the mental model and the weight settings. The next page is the recipe.

---

## Two ways to be grabbed

The object answers `GetGrabMode` with an `E_GrabMode`:

| Mode | What happens | Shipped examples |
|---|---|---|
| `Free` | The object is a physics body pulled toward the point in front of your eyes. It bumps into walls, swings and lags behind when it is heavy | every `BP_Loot_*` |
| `Axis` | The object moves along one hinge or one rail only. Your hand drives a 0 to 1 value | `BP_LeverBase`, `BP_SlidingDoor_Single` |

In `Free` mode you hold the object by the exact point you aimed at. It does not jump to your hands and it keeps its own rotation, unless it asks to be held upright (see below). A bat grabbed by the handle is carried by the handle.

`Axis` objects are built from `BP_GrabAxisBase` and a `BP_GrabAxisComponent`, covered in [Make your own door, switch or lever](../interaction/make_your_own_door_or_lever.md).

---

## The interface on the object

| Function | What the object does in it |
|---|---|
| `CanGrab` | Returns true when the `Grabber` may take it now. Loot returns false while someone else holds it |
| `GetGrabMode` | Returns `Free` or `Axis` |
| `Grab` | Runs on the server. Remember who holds you |
| `Release` | Runs on the server, and on the carrier's machine the moment the hand opens. Forget the holder |
| `UpdateGrabAim` | `Axis` only. Receives where the hand is, every frame, on the carrier's machine |
| `GetGrabMotionComponent` | The component that moves. Return nothing and the component that was hit is used |
| `GetGrabUpright` | `Free` only. Whether to hold the object upright, and how (see below) |
| `GetGrabHolder` | Returns who holds the object. A melee weapon needs it to know who swings |

---

## The component on the player

`BP_GrabComponent` sits on `BP_FriendslopCharacter`. It traces from the eyes, tells the HUD what can be grabbed, and carries the object with a Physics Handle it creates by itself at start. You drop one component on a pawn, nothing else.

While you hold something, the mouse wheel pushes it away or pulls it closer (`IA_ScrollSlot`, the same action that cycles inventory slots when you hold nothing). Push it too far and it drops.

There is no throw: releasing the button drops the object.

The HUD cursor, `WBP_GrabHand` inside `WBP_Gameplay`, shows an open hand on what you can grab and a closed hand on the grab point while you hold. It finds the component by itself. For a cursor of your own, call `GetHandState`: it returns `Visible`, `World Location` and `Closed`, which is all a grab cursor needs. See [The HUD, and adding to it](../ui/the_hud.md).

---

## Weight is the mass

There is no weight field. The weight of an object is the mass of its physics body: `Mass (kg)` in the **Physics** section of the mesh, with its override ticked.

Up to `Reference Mass Kg` (`5`) an object follows your aim at full speed. Above it, the hand target slows down in proportion to the mass, so heavy objects trail behind you. The `BP_Loot_GoldBar` mesh is `12.5` kg, the `BP_Loot_BaseballBat` `1` kg. The `180` kg `BP_Loot_EngineBlock` is too heavy to leave the ground.

To make one object feel heavier or lighter, change its mass. To change how all objects feel, change `Reference Mass Kg` or `Interpolation Speed` on the component. Do not reach for the stiffness to give weight: the physics engine scales it by the mass, so a heavier object does not feel heavier that way.

---

## Holding an object upright

A bat carried by its handle would hang down. `GetGrabUpright` lets the object ask to be held upright instead, and to lean with your speed. On `BP_LootBase` these are three fields in `Settings|Grab`:

| Field | What it does | Shipped on the bat and the hammer |
|---|---|---|
| `Align Upright` | Hold the object upright while carried | `true` |
| `Upright Offset` | The rotation that makes your mesh stand up. A mesh whose handle points along +X needs a `Pitch` of `-90` | `0, 0, 0` |
| `Max Lean Angle` | How far it leans, in degrees, at full carrier speed | `45` |

While such an object is held, the handle turns it straight toward its upright pose plus the lean, on a softer rotation spring: `Upright Angular Stiffness` and `Upright Angular Damping` take the place of `Angular Stiffness` and `Angular Damping` (see below). Any other object keeps the stiff default. The choice is made each time an object is grabbed.

---

## The player side, in Settings|Grab

All on `BP_GrabComponent`:

| Field | What it does | Shipped default |
|---|---|---|
| `Trace Distance` | How far you reach, in cm | `250` |
| `Trace Channel` | The channel the trace uses | `Visibility` |
| `Break Distance` | How far, in cm, the grab point may get beyond the distance you grabbed it at before the object drops | `100` |
| `Interpolation Speed` | How fast the hand target follows your aim for a light object | `50` |
| `Reference Mass Kg` | The mass above which objects start to lag | `5` |
| `Linear Stiffness` | Spring of the handle, on position | `750` |
| `Linear Damping` | Damping of the handle, on position | `200` |
| `Angular Stiffness` | Spring of the handle, on rotation, for an object not held upright | `1500` |
| `Angular Damping` | Damping of the handle, on rotation, for an object not held upright | `1500` |
| `Lean Reference Speed` | Carrier speed, in cm/s, at which an upright object reaches `Max Lean Angle` | `300` |
| `Lean Interp Speed` | Smoothing of the lean | `20` |
| `Upright Angular Stiffness` | Spring of the handle, on rotation, while the held object returns `Align Upright` true | `250` |
| `Upright Angular Damping` | Damping of the handle, on rotation, while the held object returns `Align Upright` true | `35` |
| `Hold Distance Step` | Distance per mouse wheel notch, in cm | `15` |
| `Min Hold Distance` | The closest the wheel can pull an object, in cm | `40` |

`Server Distance Tolerance` (`1.5`, in `Settings|Network`) lets the server accept a grab from up to `Trace Distance` times this value.

What does not exist: a carry limit, a slow down while carrying, and carrying one object with two players. Only one player can hold an object at a time.

---

## Who moves a carried object

The player who holds an object simulates it on their own machine, so the hold and the throw answer at once. Their machine sends the object's position to the server 60 times a second, and every other machine, the server included, replays it a little behind. After a throw, the thrower's machine keeps the object until it comes to rest, then the server takes it back.

The grab adds a `BP_GrabDriveComponent` to an object the first time someone takes it, so there is nothing to add to your own actors. Its settings are in the Class Defaults of `BP_GrabDriveComponent`, in `Settings|Drive`:

| Field | What it does | Default |
|---|---|---|
| `Interp Delay` | How far behind, in seconds, the other machines replay the object. Lower follows closer, and shows a late packet as a jump | `0.1` |
| `Send Rate` | Positions sent to the server per second while a player drives the object | `60` |
| `Rest Speed` | Below this speed, in cm/s, the object counts as still | `5` |
| `Rest Seconds` | How long it stays still before the server takes it back | `0.3` |
| `Max Drive Distance` | The farthest, in cm, the server lets a driven object be from the player who drives it | `3000` |

---

## The dispatchers

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnGrabbed` | This player took an object. Gives `Target` | every player |
| `OnReleased` | This player let go | every player |
| `OnGrabTargetChanged` | The aimed grabbable changed. Gives `Target`, empty when nothing is aimed | owning player |

The template character uses `OnGrabbed` and `OnReleased` to lock the camera on a lever or a sliding door while it is held. On your own character that is yours to wire: see [How the player character works](../player/how_the_player_works.md#your-own-character-blueprint).

---

## Where to go next

- [Make any actor grabbable](make_an_actor_grabbable.md)
- [Turn any grabbable object into a melee weapon](../weapons/make_a_melee_weapon.md)

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
