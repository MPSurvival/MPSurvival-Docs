# Sounds, footsteps and surfaces

Every sound in the template is a Sound Cue that carries its own Sound Class, and every sound heard from a place in the world also carries its own attenuation. Footsteps come from one component you can put on any pawn, and they pick a sound from the surface under the foot.

- The sounds: `Content/MPFriendslop/Audios/`
- The footstep component: `Content/MPFriendslop/Blueprints/ActorComponents/BP_FootstepComponent`
- The footstep notify: `Content/MPFriendslop/Blueprints/AnimNotifies/BP_FootstepNotify`

---

## Add a sound

1. Import your wave into the folder that fits: `Audios/Effects/`, `Audios/Widgets/` or `Audios/Footsteps/<Surface>/`.
2. Right click the wave, then **Create Cue**. Name it `CUE_<Name>`.
3. Open the cue. In its Details panel, set `Sound Class`: `SCLS_Effects` for anything heard in the world, `SCLS_UI` for menus, `SCLS_Music` for music.
4. For a sound heard in the world, set `Attenuation Settings` to one of the `ATT_` assets below.
5. Save, and play the cue from your Blueprint. Leave the attenuation pin of the play node empty: a value there replaces the one on the cue.

| Attenuation | Used by |
|---|---|
| `ATT_Footstep` | Player footsteps |
| `ATT_Impact` | Loot impacts, breaks, melee swings and hits, the flashlight switch |
| `ATT_Weapon_Fire` | The shotgun |
| `ATT_Warden_Footstep` | The Warden's stomps |
| `ATT_Warden_Laser` | The Warden's laser loop |

!!! warning
    A cue with no `Sound Class` still plays, but the volume sliders of the settings menu never touch it. A world cue with no `Attenuation Settings` plays flat in both ears at full volume, wherever it comes from. Neither case logs anything.

The sliders and the classes they drive are on [Settings and volume sliders](../ui/settings_and_audio.md).

---

## How footsteps are played

`BP_FootstepComponent` sits on the player character. It traces straight down from where the step happens, reads the surface it hits, and plays the matching sound attached to the pawn.

Two sources call it:

- **Your own steps** follow the camera stride of `BP_ViewMotionComponent`, so you hear yourself in every gait, walking, running and crouching. The stride is tuned on [Movement, stamina and the camera](../player/movement_stamina_and_camera.md).
- **Everyone else's steps** come from `BP_FootstepNotify` on the animations. A pawn with no `BP_ViewMotionComponent`, such as an enemy or your own third person character, always uses the notifies. There is nothing to switch.

The landing goes through the same component, and `Land Montage` plays on every machine. You hear your own landing as a step played by `BP_ViewMotionComponent`, at its `Landing Volume Scale` (`1.8`, in `Settings|Footsteps`). Everyone else hears the landing notify of the animation.

| Field | What it does | Shipped |
|---|---|---|
| `Footstep Default` | Played when the floor has no surface | `CUE_Footstep_Concrete` |
| `Footstep Concrete` | Surface `Concrete` | `CUE_Footstep_Concrete` |
| `Footstep Ground` | Surface `Ground` | `CUE_Footstep_Ground` |
| `Footstep Metal` | Surface `Metal` | `CUE_Footstep_Metal` |
| `Footstep Water` | Surface `Water` | `CUE_Footstep_Water` |
| `Footstep Wood` | Surface `Wood` | `CUE_Footstep_Wood` |
| `Footstep Volume` | Volume of every step, multiplied by the notify's `Volume Scale` | `1.0` |
| `Footstep Trace Cm` | How far down the trace looks for a floor | `120` |
| `Land Montage` | Played on landing, on every machine | `AM_Land` |

All of them are in `Settings|Footsteps`. Leaving a sound empty is allowed and simply means that surface is silent.

---

## Give a floor a surface

The project declares five surfaces: `Concrete`, `Ground`, `Metal`, `Water` and `Wood`. A floor only sounds like one of them once it has a Physical Material that says so. The template ships one per surface, in `Content/MPFriendslop/Materials/Environments/PhysicalMaterials/`:

| Physical Material | Surface Type |
|---|---|
| `PM_Concrete` | `Concrete` |
| `PM_Ground` | `Ground` |
| `PM_Metal` | `Metal` |
| `PM_Water` | `Water` |
| `PM_Wood` | `Wood` |

The floors of the modular kit, `SM_Mod_Floor_*`, already use `PM_Metal`. To give your own floor a surface:

1. Open the floor's Static Mesh, for example `SM_Mod_Floor_400`, and set `Simple Collision Physical Material` to the `PM_` of its surface. Every copy of that mesh in every level now sounds like it.
2. For one placed floor only, select it in the level and set `Phys Material Override` on its component instead.

The footstep trace hits the simple collision, so the Physical Material has to be on the collision, as in steps 1 and 2. A floor with none plays `Footstep Default`.

---

## Put footsteps on your own animations

1. Open your walk or run cycle.
2. On a notify track, at the frame where a foot touches the ground, right click, then **Add Notify**, then `BP_FootstepNotify`.
3. Set `Foot Socket` to the bone or socket of that foot: `foot_l` or `foot_r`.
4. Repeat for every contact of the cycle.

| Field | What it does | Default |
|---|---|---|
| `Foot Socket` | Where the trace starts and where the sound is heard | `foot_l` |
| `Volume Scale` | Multiplies `Footstep Volume` for this step only | `1` |

The notify looks for `BP_FootstepComponent` on the pawn playing the animation. A pawn without the component stays silent. The shipped walk and run cycles `AS_Robot_Walk_*` and `AS_Robot_Run_*` in `Demo/Animations/Robot/Standing/` and `AS_Robot_Land` in `Demo/Animations/Robot/Airborne/` are a ready example to copy the contact frames from. The first notify of `AS_Robot_Land` uses a `Volume Scale` of `1.8`.

---

## React to a step

`BP_FootstepComponent` calls `OnFootstep` on every audible step. Bind it for dust, decals or a camera shake.

| Dispatcher | Gives | Fires on |
|---|---|---|
| `OnFootstep` | `Surface`, `Location` | every player |

Footstep sounds are not what enemies hear. The Warden listens to `BP_NoiseComponent`, which counts distance on the server. See [How the enemy AI works](../ai/how_the_enemy_ai_works.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
