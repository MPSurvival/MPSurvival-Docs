# How the player character works

The player is `BP_FriendslopCharacter`, a child of `Character`, driven by `BP_FriendslopPlayerController`. The character has no variables of its own. Everything it does is a component, and each component can go on your own pawn too.

- The character and its controller: `Content/MPFriendslop/Blueprints/PlayerCharacter/`
- The components: `Content/MPFriendslop/Blueprints/ActorComponents/`

This page is the map. The pages after it are the recipes.

---

## The controller

`BP_FriendslopPlayerController` receives every gameplay key. It handles the five that belong to any pawn itself: move, look, jump, crouch and the pause menu. It only uses `Pawn` and `Character` functions for them, so it drives any `Character` you give it.

When the local player starts, it adds its mapping context and creates the HUD. These are its fields:

| Field | What it does | Shipped value |
|---|---|---|
| `Default Context` | The Input Mapping Context added for the local player, at priority 0 | `IMC_Default` |
| `Gameplay Widget Class` | The in-game HUD | `WBP_Gameplay` |
| `Recap Widget Class` | The end of run recap screen | `WBP_RunRecap` |
| `Pause Widget Class` | The menu shell opened on pause | `WBP_MenuRoot` |
| `Pause Page Class` | The page shown inside it | `WBP_PausePage` |
| `Loading Widget Class` | The loading screen, shown until the clock starts and the level is visible | `WBP_LoadingScreen` |

Every other key is passed to the component that owns the system, on the pawn it controls. The controller finds it with `Get Component by Class` and calls one of its functions. A pawn without that component, such as the spectator, simply ignores the key. The components read no key themselves, so they work on your own pawn with nothing to wire.

| Action | Component | Functions called |
|---|---|---|
| `IA_Interact` | `BP_InteractionComponent` | `BeginInteraction`, `UpdateHold`, `ReleaseInteraction` |
| `IA_Sprint` | `BP_StaminaComponent` | `StartSprint`, `StopSprint` |
| `IA_Grab` | `BP_GrabComponent` | `TryGrab`, `TryRelease` |
| `IA_ScrollSlot` | `BP_GrabComponent`, then `BP_InventoryComponent` | `AdjustHoldDistance`, `CycleSlot` |
| `IA_UseItem`, `IA_Reload` | `BP_InventoryComponent` | `PressUse`, `ReleaseUse`, `PressReload` |
| `IA_DropItem`, `IA_SelectSlot` | `BP_InventoryComponent` | `Server_DropSelected`, `SelectSlotFromAxis` |
| `IA_Ping`, `IA_PingDanger` | `BP_PingComponent` | `RequestPing`, `RequestDangerPing` |
| `IA_Emote`, `IA_EmoteWheelPage` | `BP_EmoteComponent` | `OpenWheel`, `CommitWheelPick`, `StepWheelPage` |

The one exception is the shop screen pointer: `BP_ShopPointerComponent` reads `IA_Interact` itself. To change a key, change it in `IMC_Default`. See [The controls, and adding an input](../start/controls.md).

---

## What the character is made of

| Component | What it does | Page |
|---|---|---|
| `ViewRoot` > `ViewCamera` | The first person camera, at eye height | Movement, stamina and the camera |
| `BP_ViewMotionComponent` | Head bob, sway, lean, landing dip, crouch smoothing, sprint FOV | same page |
| `BP_StaminaComponent` | Sprint, and what sprinting and jumping cost | same page |
| `BP_VitalsSystem` | Health and stamina values, damage and death | Health, damage and new vitals |
| `BP_DamageFeedbackComponent` | The hurt sound, camera shake and red screen edge on your own machine. The edge is `WBP_DamageVignette` in the HUD, bound to this component's `OnDamageTaken` | same page |
| `BP_FootstepComponent` | Footsteps and landing sounds | [Sounds, footsteps and surfaces](../world/sounds_and_footsteps.md) |
| `BP_NoiseComponent` | The noise enemies hear | [How the enemy AI works, and what it hears](../ai/how_the_enemy_ai_works.md) |
| `BP_InteractionComponent` | Looks for what you aim at and uses it | [How interaction works](../interaction/how_interaction_works.md) |
| `BP_GrabComponent` | Picks up and carries physics objects, pulls levers | [How grabbing works, and tuning the weight](../grab/how_grabbing_works.md) |
| `BP_InventoryComponent`, `BP_ConsumableComponent` | The item slots, and items that restore a vital | [How items and the inventory work](../items/how_items_work.md) |
| `BP_ShopComponent`, `BP_ShopPointerComponent` | Buying from the shop terminal | [Add an item to the shop](../items/the_shop.md) |
| `BP_DeathComponent` | What happens when you die | [Death, revive and the spectator](../death/how_death_works.md) |
| `BP_EmoteComponent`, `Face` (a `BP_FaceComponent`) | Emotes, and the screen face | [Add an emote or a face](../crew/add_an_emote_or_a_face.md) |
| `BP_CosmeticVisualsComponent` | Draws the hat, accessory and pattern | [How cosmetics work](../crew/how_cosmetics_work.md) |
| `BP_PlayerColorTintComponent`, `BP_NameplateComponent`, `BP_PingComponent` | Player colour, the name above the head, pings | [Player colours, nameplates and pings](../crew/colours_nameplates_and_pings.md) |

Three more components are not on the pawn: `BP_CosmeticComponent` and `BP_PlayerColorComponent` on `BP_FriendslopPlayerState`, and `BP_SpectatorComponent` on the controller.

The character also implements `BPI_VitalManagerInterface`. Other systems reach the vitals through its `GetVitalComponent`, never by casting to the character.

---

## The native values

Walking, crouching and jumping are plain Character Movement. The template only adds sprint on top.

| Where | Field | Shipped value |
|---|---|---|
| Character Movement | `Max Walk Speed` | `200` |
| Character Movement | `Max Walk Speed Crouched` | `130` |
| Character Movement | `Jump Z Velocity` | `420` |
| `BP_StaminaComponent` | `Sprint Speed` | `500` |
| `ViewCamera` | `Field Of View` | `90` |

The eyes sit 168 cm above the floor: `ViewRoot` is 80 cm above the centre of an 88 cm half height capsule. `Crouched Eye Height` is also 80, on purpose, so traces that start from the eyes stay right when crouched.

The body is `SKM_Robot_Courier`, animated by `ABP_FriendslopLocomotion`. The local player never sees it: `Owner No See` is on and `Cast Hidden Shadow` is off. The other players see the body and its shadow.

---

## Put it on your own character

There are two ways in:

- **Keep the template's character, change the body.** A child of `BP_FriendslopCharacter` with your mesh. Nothing is lost.
- **Your own Character Blueprint.** You add the components you want. Each one works on its own, and a system you leave out is not there.

### Keep the template's character, change the body

1. Right click `BP_FriendslopCharacter` and pick **Create Child Blueprint Class**.
2. In the child, select `Mesh` and set `Skeletal Mesh Asset` to your mesh.
3. Leave `Anim Class` on `ABP_FriendslopLocomotion` if your mesh uses `SKEL_Mannequin`, the UE5 Manny skeleton.
4. Open `BP_FriendslopGameMode` and set `Default Pawn Class` to your child.

Your mesh has to carry the names the systems look for:

| Name on your mesh | Used by | On `SKEL_Mannequin` |
|---|---|---|
| `FlashlightSocket` | The flashlight in hand | On the skeleton, you get it for free |
| `ShotgunSocket` | The shotgun in hand | Only on `SKM_Robot_Courier`. Add it to your mesh, in the same place |
| `HatSocket` (bone `head`), `BackSocket` (bone `spine_04`) | Hats and accessories | On the skeleton, you get them for free |
| `FaceScreen` (bone `head`) | The `Face` screen | On the skeleton, but not where the robot's screen is. Add `FaceScreen` to your mesh, on your character's face |
| Bone `head` | Where the head appears on death | Manny bone |
| Material slot named `M_Robot_Shell` | Player colour and pattern | Name your slot this way, or change `Tinted Slot Names` on `BP_PlayerColorTintComponent` |

A socket on the mesh wins over a socket of the same name on the skeleton. To move a hat, an accessory or the face on the robot, edit the socket on `SKM_Robot_Courier`, not on `SKEL_Mannequin`.

### Your own Character Blueprint

The parent must be `Character`: the camera motion, stamina and landing all read Character Movement.

1. Under the capsule, add a Scene Component at eye height, and a Camera under it, like `ViewRoot` and `ViewCamera`.
2. On the camera, turn `Use Pawn Control Rotation` off. `BP_ViewMotionComponent` writes the camera rotation itself.
3. Set `Base Eye Height` and `Crouched Eye Height` to the height of that Scene Component.
4. On your body mesh, turn `Owner No See` on and `Cast Hidden Shadow` off.
5. Add the components you want, from the table in [What the character is made of](#what-the-character-is-made-of). `BP_ViewMotionComponent` needs `Motion` set to `DA_ViewMotion_Default`, or it does nothing.
6. Add the interface `BPI_VitalManagerInterface`. Make `GetVitalComponent` return your `BP_VitalsSystem`, and put in `OnDeath` what your pawn does when it dies.
7. Forward the events of the table below.
8. Set `Default Pawn Class` on `BP_FriendslopGameMode` to your character. Keep `BP_FriendslopPlayerController`: it drives any `Character`.

!!! warning
    Leave `Crouched Eye Height` at the engine default and the camera still looks right, but every trace that starts from the eyes starts from the wrong height when you crouch. Interaction and grab then aim lower than the camera, with no error.

These are the whole event graph of `BP_FriendslopCharacter`. Copy the ones for the components you added.

| Event on your character | Call |
|---|---|
| `On Start Crouch`, if locally controlled | `AddCrouchTransition` on `BP_ViewMotionComponent`, with `Scaled Half Height Adjust` |
| `On End Crouch`, if locally controlled | `AddCrouchTransition`, with minus `Scaled Half Height Adjust` |
| `On Landed` | `NotifyLanded` on `BP_FootstepComponent`, and if locally controlled, `ApplyLandImpact` on `BP_ViewMotionComponent` with the velocity Z |
| `On Jumped` | `NotifyJumped` on `BP_StaminaComponent` |
| `OnGrabbed` of `BP_GrabComponent` | If `IsAxisMode` is true (a lever or a door), `GetGrabAttachment`, then `LockView` on `BP_ViewMotionComponent` |
| `OnReleased` of `BP_GrabComponent` | `UnlockView` on `BP_ViewMotionComponent` |
| `NotifyRunEnded` and `NotifyArenaStarted`, from the interface `BPI_RunControl` added to your character | `EndBleedoutNow` on `BP_DeathComponent` |

`LockView` makes the camera follow the lever or the door instead of turning the body.

Three pieces do not go on the pawn: `BP_CosmeticComponent` and `BP_PlayerColorComponent` live on the PlayerState, `BP_SpectatorComponent` on the controller. To replace those classes too, see [Use your own game framework classes](../ui/use_your_own_framework.md). The HUD needs nothing: each piece of `WBP_Gameplay` looks for its component on the possessed pawn, and stays idle when it is missing.

---

## Use your own animations

`ABP_FriendslopLocomotion` only casts its owner to `Character`, so it runs on any character. Its two fields are in `Settings|Animation`: `Standing Blend Space` (`BS_Locomotion_Standing`) and `Crouching Blend Space` (`BS_Locomotion_Crouching`).

On `SKEL_Mannequin`:

1. Put your clips in the samples of the two blend spaces. Or make your own, and set them in the two fields.
2. For the up and down aim of the upper body, replace the samples of `BS_AimOffset_Pitch`.
3. The jump and fall clips are set inside the `Jump` and `Fall` states of the `Locomotion` state machine.
4. Add `BP_FootstepNotify` at each foot contact of your walk and run cycles, with `Foot Socket` set to `foot_l` or `foot_r`. Other players hear your steps only where a clip carries it.

Keep the speeds in step with your clips or the feet slide: walk at `200`, run at `500`, crouch at `130` (the crouch clips play at a rate of `0.65`).

If you write your own AnimBP instead, keep two slots: `DefaultSlot` after the locomotion, for `AM_Land` and any full body montage, and `UpperBody`, for the emotes and the third person reload. Implement `BPI_HandIKSuspend` as `ABP_FriendslopLocomotion` does: count `PushHandIKSuspend` and `PopHandIKSuspend`, and fade the hand IK out while the count is above zero. Without it, the left hand stays glued to the grip during reloads and emotes. What a held item gives the AnimBP is in [Make a held item, its hold pose and hand IK](../items/make_a_held_item.md).

For another skeleton, retarget the animations with Unreal's IK Rig and IK Retargeter, or build a new AnimBP on your skeleton. The robot's clips come from an automatic retarget, kept in `Demo/Retargeting/`. Every name of the socket table then has to exist on your skeleton or your mesh.

---

## Where to go next

- [Movement, stamina and the camera](movement_stamina_and_camera.md) to tune speeds, sprint and the camera feel.
- [Health, damage and new vitals](health_and_damage.md) to hurt, heal and add a vital.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
