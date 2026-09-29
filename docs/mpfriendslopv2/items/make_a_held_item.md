# Make a held item, its hold pose and hand IK

By the end of this page you have an item that appears in the player's hand when its slot is selected. It does its own thing when used. Other players see it held with a proper pose, with the left hand on the grip if it needs one.

If you do not have the item's Data Asset yet, start with [Add an inventory item](add_an_item.md). This page picks up from its `Wield Actor Class` field.

- The held actors: `Content/MPFriendslop/Blueprints/Environments/Items/Wieldable/`
- Their components: `Content/MPFriendslop/Blueprints/ActorComponents/`
- The item Data Assets: `Content/MPFriendslop/Blueprints/DataAssets/Item/Childs/`

---

## How a held item works

When a slot is selected, the server spawns the item's `Wield Actor Class`, usually a child of `BP_WieldedItem`, and attaches it to the player's hand. The held actor carries two copies of the item mesh:

- `ViewMesh`, the first person copy. Only its owner sees it, and it follows the camera.
- `HandMesh`, the third person copy. Everyone except its owner sees it, and it rides the hand socket.

Both meshes are set on the held actor itself, so its Blueprint viewport shows the item and you place it there. Behaviour is added with components on the held actor: the flashlight has `BP_FlashlightComponent`, the shotgun has `BP_WeaponComponent`. `BP_WieldedMedkit` and `BP_WieldedBattery` have no graph and no component of their own, because their effect lives on the pawn: they only carry their mesh and its first person placement.

---

## Step 1, make the held actor

Pick the shipped one closest to yours.

| Start from | Take this one when | What it brings |
|---|---|---|
| `BP_WieldedMedkit` | The item is a static mesh and needs nothing but to be held | Both meshes, the sway |
| `BP_WieldedFlashlight` | The item gives light and drains a charge | A `Light` spot light, `BP_LightComponent`, `BP_FlashlightComponent` |
| `BP_WieldedShotgun` | The item is a gun, or any skinned mesh that plays its own montages | See [Add a new gun](../weapons/add_a_new_gun.md) |

1. In `Childs/`, duplicate the one you picked. A child of `BP_WieldedItem` works too.
2. Name it `BP_Wielded<YourItem>` and keep it in `Childs/`.
3. Select `HandMesh`, then `ViewMesh`, and set the **Static Mesh** of both to your item's mesh.
4. Add your own components to it.
5. Open your `DA_Item_*` and set `Wield Actor Class` to it.

### Place it in first person

Where the item sits in front of the camera is the transform of `ViewMesh` in the held actor. Select `ViewMesh` in the Blueprint viewport and move it with the gizmo, or type the values in **Details**.

Read it as a position from the player's eye, in cm: X forward, Y right, Z up. The Blueprint's origin stands for the camera. A yaw of `90` points a mesh modelled along its Y axis forward.

| Item | Location | Rotation |
|---|---|---|
| Flashlight, battery | `(35, 18, -19)` | yaw `90` |
| Shotgun | `(30, 18, -19)` | yaw `90` |
| Medkit | `(50, 16, -26)` | yaw `90` |

The medkit sits further away and lower because it is a wide box: at the flashlight's place it would fill a quarter of the screen. Compile the Blueprint, then take the item in hand to judge it in game.

The third person copy is placed by the hand socket instead (step 3).

### Your own actor instead of a child

`Wield Actor Class` accepts any actor. The only thing the inventory requires of the held actor is the `BPI_Wieldable` interface, so an actor with another parent works too:

1. Open your actor, then **Class Settings**, and add `BPI_Wieldable` to **Implemented Interfaces**.
2. In **Class Defaults**, tick `Replicates`.
3. Implement the `SetWieldedItem` event. Its `Item Data` input is the item's Data Asset, sent right after the actor spawns.
4. If your actor needs the Data Asset later, store `Item Data` in a replicated variable: `SetWieldedItem` runs on the server only. `BP_WieldedItem` keeps it for hand IK.
5. Put your meshes on the actor itself.

Such an actor gets none of what `BP_WieldedItem` does for you: the two meshes, the first person copy on the camera, the sway. Hand IK also comes from `BP_WieldedItem`. To keep it, implement `BPI_HandIK` and copy `GetHandIKGrips` from `BP_WieldedItem` (step 4).

---

## Step 2, give it behaviour

Gameplay runs on the server. Your component finds `BP_InventoryComponent` on the player and binds its `On Item Used` dispatcher behind a `Has Authority` check. `BP_FlashlightComponent` does exactly this, so open it and copy the shape.

For an item with a charge (battery, ammo), the charge lives in the inventory slot, not on the held actor:

| Function on `BP_InventoryComponent` | What it does | Call it on |
|---|---|---|
| `ConsumeSelectedCharge` | Takes `Amount` off the selected slot's charge | server |
| `RefillSelectedCharge` | Fills the selected slot back to `Max Charge` | server |
| `GetSelectedChargeRatio` | Returns the charge from 0 to 1. An item with `Max Charge` at 0 returns 1 | any machine |

The dispatchers a held item can use:

| Dispatcher | On | When | Fires on |
|---|---|---|---|
| `On Item Used` | `BP_InventoryComponent` | The player used the selected item | every player |
| `On View Attached` | `BP_WieldedItem` | `ViewMesh` has been attached to the camera | owning player |

A child binds `On View Attached` for its own setup. The flashlight uses it to hook its light onto the first person mesh.

!!! warning
    `On Item Used` fires on every machine. A listener that changes the game (a charge, a door, damage) without checking `Has Authority` runs once per player, and nothing tells you. Cosmetic effects, like a sound, can listen everywhere.

---

## The flashlight, as a model

`BP_FlashlightComponent` toggles the light on use, drains the charge while it is lit, blinks when the charge is low and turns off at 0.

| Field | What it does | Shipped |
|---|---|---|
| `Drain Per Second` | Charge taken per second while lit. At 1, a charge of 100 lasts 100 seconds | `1` |
| `Low Charge Threshold` | Below this share of the charge, the light blinks | `0.2` |
| `Turn On Sound` | Played when the light turns on | `CUE_Flashlight_Switch` |
| `Turn Off Sound` | Played when the light turns off | `CUE_Flashlight_Switch` |

The beam itself is the `Light` spot light on `BP_WieldedFlashlight`. Its brightness and blink are on the `BP_LightComponent` next to it (`Strength`, `Blinking Interval`, `Blink Randomness`). `Strength` is applied when play starts, so the editor viewport does not preview it. More on that component in [Place, switch and make lamps](../world/lamps_and_lights.md).

---

## First person sway

`BP_WieldSwayComponent` moves the first person mesh when you look around, walk, strafe, jump and land. It only runs on the owning player's machine.

| Field | What it does | Shipped |
|---|---|---|
| `Look Sway Gain` | How much the item tilts when you turn your head | `9` |
| `Look Offset Cm` | How far the item slides sideways when you turn your head | `3.5` |
| `Lean Gain` | How much the item reacts when you speed up or strafe | `8` |
| `Walk Bob Cm` | Height of the walking bob, in cm | `1.2` |
| `Vertical Gain` | How much the item moves on a jump, a fall and a landing | `0.8` |
| `Sway Smoothing` | How fast the item follows its target, not how far it moves. Higher is snappier | `16` |

---

## Step 3, the hold pose

Other players see the item in the character's hand, with the upper body in a pose made for it. Apart from the socket, all of it is set on the item's Data Asset.

1. On your character's skeleton or mesh, add a socket on `hand_r` where the grip should sit. Name it after the item.
2. In the Data Asset, set `Wield Socket Name` to that socket.
3. Set `Overlay Pose` to an upper body pose holding the item.
4. If the item is aimed, set `Overlay Aim Offset` to a 1D aim offset so the torso follows the view up and down.
5. If both hands hold it, tick `Overlay Uses Left Arm`.

| Field | What it does | Flashlight | Shotgun |
|---|---|---|---|
| `Wield Socket Name` | Socket the held actor snaps to. Empty uses the `Wield Socket Name` of `BP_InventoryComponent`, `hand_r` | `FlashlightSocket` | `ShotgunSocket` |
| `Wield Relative Transform` | Offset applied after the snap | identity | identity |
| `Overlay Pose` | Upper body pose while held. Empty means no pose | `AS_Robot_Flashlight_Base` | `AS_Robot_Shotgun_Base` |
| `Overlay Aim Offset` | Vertical aim on top of the pose | `BS_Flashlight_Aim` | none |
| `Overlay Uses Left Arm` | Blends the left arm too, for a two handed item | off | on |

The shotgun ships without an aim offset, so with it the torso does not follow the view pitch. Every shipped item puts the grip in the socket and leaves `Wield Relative Transform` at identity. Do the same, because you can preview a socket on the skeleton.

Two things to know about the sockets. `FlashlightSocket` is on the skeleton `SKEL_Mannequin`, so any mesh on that skeleton has it. `ShotgunSocket` is on the mesh `SKM_Robot_Courier` only: on your own character mesh, add it yourself. The held actor attaches to the first **Skeletal Mesh** component of the pawn, so the socket must be on that one.

The pose and the aim offset are blended in by `ABP_FriendslopLocomotion`. Its `Overlay Blend Speed` (`12`) sets how fast the pose blends in and out.

---

## Step 4, hand IK

A two handed item puts the left hand on its grip with IK.

1. Open the item's third person mesh and add a socket where the left hand grabs it. By default that mesh is the one on `HandMesh`. If your held actor shows another mesh, override its `GetThirdPersonMesh` function to return that mesh. The shotgun returns its `HandSkeletalMesh`, and its socket is `LeftHandGrip` on `SKM_Tool_Shotgun_SawedOff`.
2. In the Data Asset, set `Left Hand IK Socket` to that socket name.
3. For the right hand, `Right Hand IK Socket` works the same way. No shipped item uses it.

`Hand IK Blend Speed` on `ABP_FriendslopLocomotion` (`12`) sets how fast the hand goes on and off the grip.

!!! warning
    Any montage that takes the hands off the item needs `BP_HandIKSuspendNotifyState` over that stretch. Without it, the left hand stays glued to the grip while the arm plays the animation. It is already on `AM_Shotgun_Reload_TPS` and on the three `AM_Emote_*` montages. Add it to your own reload and emote montages.

---

To sell your new item in the terminal, see [Add an item to the shop](the_shop.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
