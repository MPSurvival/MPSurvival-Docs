# Install and open the project

Everything the template owns lives in one folder: `Content/TheLastTemplate`.

---

## What you need

- **Unreal Engine 5.8**. `TheLastTemplate.uproject` is bound to that version.
- Nothing else. Every plugin the template uses ships with the engine.

---

## The plugins the template uses

| Plugin | What it does here |
|---|---|
| `Motion Warping` | Moves the character during a montage so a vault ends on the real ledge and a dodge covers the real ground. `BP_PlayerCharacter` carries the `Motion Warping` component, and the traversal and dodge montages carry the warp targets. |
| `Animation Warping` | Used by `ABP_HumanoidCharacter` and by `BP_PlayerCharacter` to line the pose up with the direction the character actually moves. |
| `Animation Locomotion Library` | `Foot Placement` in `ABP_HumanoidCharacter`, which puts the feet on uneven ground, and `Distance Matching` in `ABP_LocomotionLayers`, which stops the feet sliding when the character starts and stops. |
| `Physics Control` | The physical hit reactions and the ragdolls. `BP_HitReactionManager` drives it. |

None of the four is optional if you keep the player character. Turn one off and the assets in its row stop compiling.

---

## Press Play, what you should see

`L_ShowcaseMap` is the demo level. You spawn as `BP_PlayerCharacter` with the HUD on screen, and you can move, sprint, crouch, climb, pick things up, fight and use the menu straight away. The keys are listed in [Change the controls](change_the_controls.md).

`L_MeshesShowcaseMap` is a second level that lays out the meshes that come with the project, described in [The bonus mesh kit](..

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
