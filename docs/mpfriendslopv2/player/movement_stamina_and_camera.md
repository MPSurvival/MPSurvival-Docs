# Movement, stamina and the camera

Walking, crouching and jumping are Unreal's own Character Movement. Sprint and stamina live on one component, and everything the first person camera does lives on another. This page shows which number to change for each.

- The character: `Content/MPFriendslop/Blueprints/PlayerCharacter/BP_FriendslopCharacter`
- The components: `Blueprints/ActorComponents/BP_StaminaComponent` and `BP_ViewMotionComponent`
- The camera settings: `Blueprints/DataAssets/ViewMotion/Childs/DA_ViewMotion_Default`

---

## Walk, crouch and jump

These are native fields. Open `BP_FriendslopCharacter`, select `CharMoveComp` (Character Movement) in the Components panel, and change them there.

| Field | What it does | Shipped |
|---|---|---|
| `Max Walk Speed` | Walking speed, in cm/s | `200` |
| `Max Walk Speed Crouched` | Speed while crouched | `130` |
| `Jump Z Velocity` | How high a jump goes | `420` |
| `Air Control` | How much you can steer in the air | `0.05` |
| `Crouched Half Height` | Capsule half height while crouched | `40` |

Crouch is held: `Left Ctrl` down crouches, releasing it stands up. Jump is `Space`. To change a key, see [The controls](../start/controls.md).

The body animations are authored for these speeds: walk at `200`, run at `500`, and the crouch clips are slowed to match `130`. Change a speed without changing the animations and the feet slide. Swapping the clips is covered in [Use your own animations](how_the_player_works.md#use-your-own-animations).

Character Movement has one top speed for every direction, so strafing and walking backward move as fast as walking forward.

!!! warning
    `Crouched Eye Height` on the character (Class Defaults) must stay equal to the height of `ViewRoot`, the scene component that holds the camera (`80` on both). If they differ, every trace that starts from the eyes starts from the wrong height as soon as you crouch. Nothing reports it.

---

## Sprint and stamina

Hold `Left Shift` to sprint. Sprinting drains stamina, jumping costs a fixed amount, and stamina comes back after a short pause. All of it is on `BP_StaminaComponent`, on the character: the first three fields in `Settings|Sprint`, the rest in `Settings|Stamina`.

| Field | What it does | Shipped |
|---|---|---|
| `Sprint Speed` | Top speed while sprinting | `500` |
| `Sprint Stamina Per Second` | Stamina spent per second of sprint | `20` |
| `Minimum Stamina To Sprint` | Below this, the sprint does not start | `10` |
| `Jump Stamina Cost` | Stamina spent per jump | `12` |
| `Stamina Regen Per Second` | Stamina recovered per second | `18` |
| `Regen Delay` | Seconds without spending before regen starts | `1` |
| `Update Interval` | How often, in seconds, stamina is updated | `0.1` |

Walking speed is not on this component. It reads `Max Walk Speed` from Character Movement when the game starts and puts it back when the sprint ends, so tune walking in one place only.

Sprint only drains while you are on the ground, not crouched, and actually moving. You cannot start a sprint while crouched. When stamina reaches 0 the sprint stops on its own.

The stamina value itself is a vital, stored with health on `BP_VitalsSystem`, so the pawn needs a `Stamina` row there. See [Health, damage and new vitals](health_and_damage.md).

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnSprintStarted` | The sprint begins | every player |
| `OnSprintStopped` | The sprint ends, by release or empty stamina | every player |

Bind these for a breathing sound, a HUD effect or anything else, without opening the component.

---

## Tune the camera

Head bob, look sway, lean, the landing dip, crouch smoothing and the sprint FOV kick all come from one Data Asset read by `BP_ViewMotionComponent`. Its class is `BP_ViewMotionDataAsset`, in `Blueprints/DataAssets/ViewMotion/`, and `Motion` on the component only accepts that class. `DA_ViewMotion_Default` is the one the character ships with.

1. In `Blueprints/DataAssets/ViewMotion/Childs/`, duplicate `DA_ViewMotion_Default`. Name it `DA_ViewMotion_<Name>`.
2. Open it and change the groups below.
3. Open `BP_FriendslopCharacter` and select `BP_ViewMotionComponent`.
4. In `Settings|View Motion`, set `Motion` to your asset.

The fields are grouped by category. A field ending in `Cm`, `Deg` or `Hz` is in centimetres, degrees or hertz.

| Group | Main fields | Shipped |
|---|---|---|
| Walk | `Step Length Cm`, `Full Motion Speed`, `Max Motion Scale` | `95`, `240`, `1.65` |
| Bob | `Bob Vertical Cm`, `Bob Lateral Cm`, `Bob Roll Deg`, `Bob Pitch Deg` | `1.1`, `0.8`, `0.225`, `0.125` |
| Look Sway | `Look Sway Yaw Deg`, `Look Sway Pitch Deg`, `Look Sway Roll Deg` | `1.6`, `1.2`, `-2.6` |
| Lean | `Strafe Roll Deg`, `Accel Pitch Deg`, `Lean Stiffness`, `Lean Damping` | `1.1`, `0.7`, `70`, `12` |
| Landing | `Land Dip Cm`, `Land Full Impact Speed`, `Land Stiffness`, `Land Damping` | `14`, `700`, `120`, `16` |
| Idle | `Breath Cm`, `Breath Rate Hz` | `0.7`, `0.22` |
| Sprint | `Sprint Fov Add`, `Fov Blend Speed` | `3`, `7` |
| Crouch | `Crouch Blend Speed` | `11` |
| Jump | `Air Offset Cm`, `Air Velocity Reference`, `Air Blend Speed` | `9`, `450`, `9` |

`Step Length Cm` also times your own footsteps, so a longer stride means fewer steps heard.

For a calmer camera, lower the Bob, Look Sway and Lean values. For no bob at all, set the Bob fields to `0`.

---

## Two things to know about the camera

- **The component is the only thing that moves the camera.** Every frame it writes the position of `ViewRoot`, and the rotation and field of view of `ViewCamera`. Keep `Use Pawn Control Rotation` off on the camera. Do not set these from another Blueprint: the component overwrites them on the next frame.
- **The player's field of view wins.** `ViewCamera` ships at `90`, but the **Field of View** setting of the settings menu (`90` by default) replaces it when the game starts, and again each time the player changes it. Changing the camera alone does not change what players see. See [Settings and volume sliders](../ui/settings_and_audio.md).

Leaving `Motion` empty is not a way to get a still camera. The component then never runs, and it is also what applies looking up and down to the camera. For a still camera, use a Data Asset whose Bob, Look Sway, Lean, Landing, Idle and Jump amounts are `0`, and whose `Sprint Fov Add` is `0`.

Other components read the camera motion through `BPI_ViewMotion`, which `BP_ViewMotionComponent` implements. Its one function, `GetViewMotionState`, returns an `S_ViewMotionState`, which holds the current look sway, lean, stride phase and vertical offset. `BP_WieldSwayComponent` uses it to sway the held item with the camera. It looks on the pawn for a component that implements `BPI_ViewMotion`, so it never needs `BP_ViewMotionComponent` by name. To make your own effect follow the camera, do the same with `Get Components by Interface` on the pawn. Leave `BP_ViewMotionComponent` off your own pawn and the item in your hands stays still in your view, with no error.

---

To show stamina and health on screen, see [The HUD, and adding to it](../ui/the_hud.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
