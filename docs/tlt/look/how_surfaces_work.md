# How surfaces work

Every impact asks the same question: what did I just hit? The answer is a **surface**, one label carried by the mesh you shot or by the floor you stepped on. Bullet effects, decals and footstep sounds all read that one label, so setting it once on a mesh makes all three correct at the same time.

This page tells you where the label lives, who reads it, and what it picks.

---

## The surfaces that ship

Six surface types are declared in the project, in the `SurfaceType1` to `SurfaceType6` slots, plus the engine `Default`. Each one has a Physical Material in `Content/TheLastTemplate/Materials/PhysicsMaterials/`.

| Surface | Physical Material | Note |
|---|---|---|
| `Default` | `PM_Default` | what you get when nothing sets a surface |
| `Metal` | `PM_Metal` | used by `MI_Prop_Metal` and `MI_Prop_SteelPainted` |
| `Wood` | `PM_Wood` | used by `MI_Prop_Wood` |
| `Grass` | `PM_Grass` | |
| `Blood` | `PM_Blood` | used by `M_CharacterColor` |
| `Glass` | `PM_Glass` | |
| `Water` | `PM_Water` | |

`L_ShowcaseMap` has a wall of six panels, one per surface, so you can fire at each of them and compare.

---

## Where the label goes

Two fields can carry a physical material, and they do not behave the same.

- On a material or a material instance: the `Phys Material` field.
- On a placed mesh component: the `Phys Material Override` field, under **Collision**.

!!! warning "A physical material on a material is invisible to a simple trace"
    A physical material set on a material is only read when the trace runs against per polygon collision. Simple collision, which is what a mesh uses by default, never sees it, and gunfire traces against collision. So a mesh whose surface only lives on its material still gives you the default impact. Put the `PM_*` on the mesh component's `Phys Material Override` when you want bullets to react. The six panels in `L_ShowcaseMap` are set up that way, not through their materials.

---

## What reads a surface

Two assets, and only two.

| Asset | When it looks | What it picks |
|---|---|---|
| `BP_WeaponManager` | a bullet lands | impact effect, decal, impact sound |
| `AN_FootstepSound` | a foot lands | footstep sound, and the noise the AI hears |

- `Content/TheLastTemplate/Blueprints/ActorComponents/BP_WeaponManager`
- `Content/TheLastTemplate/Animations/AnimNotifies/Character/AN_FootstepSound`

Nothing else in the template reads the surface. Melee, explosions and throwables do not change with what you hit. A punch plays the sound in the `Impact Sound` field of `BP_CombatComponent` at the bone it connected with, whatever that bone is made of. See [How melee works](../melee/melee_how_it_works.md).

---

## What a surface picks

Three sets of assets, one per stage.

| Stage | Folder | What ships |
|---|---|---|
| Impact effect | `Content/TheLastTemplate/Niagara/Impacts/` | `NS_Impact_Blood`, `NS_Impact_Concrete`, `NS_Impact_Grass`, `NS_Impact_Metal`, `NS_Impact_Water`, `NS_Impact_Wood` |
| Decal | `Content/TheLastTemplate/Materials/Effects/Decals/Instances/` | `MI_Decal_Impact_Blood`, `MI_Decal_Impact_Concrete`, `MI_Decal_Impact_Dirt`, `MI_Decal_Impact_Glass`, `MI_Decal_Impact_Metal`, `MI_Decal_Impact_Wood` |
| Sound | `Content/TheLastTemplate/Audios/Footsteps/` | `SC_FS_Concrete`, `SC_FS_Grass`, `SC_FS_Ground`, `SC_FS_Metal`, `SC_FS_Water`, `SC_FS_Wood` |

Read those three lists carefully, because they are not a clean grid and the names lie a little. `NS_Impact_Concrete` and `MI_Decal_Impact_Dirt` have no surface of the same name. `Glass` has a decal but no Niagara system of its own. `AN_FootstepSound` wires five of the six sound cues, leaving `SC_FS_Grass` out, while `BP_WeaponManager` wires a different five and leaves `SC_FS_Ground` out. The file names are not the wiring. The switch inside those two assets is, so open one and follow your surface through it before you assume anything.

Bullet impacts and footsteps share the same sound cues. Changing `SC_FS_Metal` changes both.

The six decal instances are all children of `M_Decal_Impact` and share two 4 by 4 atlas textures, `T_Decal_Impact_MRA_4x4` and `T_Decal_Impact_N_4x4`. They differ mostly by `Atlas Row`, which picks the cell, and by `Surface Tint`, `Opacity Mul` and `Rough Damaged`. The six Niagara systems are built on four shared emitters, `NE_Debris`, `NE_Dust`, `NE_Flash` and `NE_Spark`, and expose `User.ImpactStrength`, `User.DustTint`, `User.DebrisTint` and `User.FreshBreak`.

---

## Footsteps also make noise the AI hears

`AN_FootstepSound` does two jobs on the same beat. It plays the cue for the surface, and it reports a noise event to the AI perception system. The audible range is read from two fields on the notify, `Walk Noise Range` and `Run Noise Range`, and the current gait decides where between them the step lands. A sprinting player is heard from much further than a walking one.

What you place on an animation is never `AN_FootstepSound` itself. It is one of its two children, `AN_FootstepSound_L` and `AN_FootstepSound_R`, which do nothing but say which foot it was through their `Set Left Foot Up` field. All the work, and both range fields, live on the parent. Change them there and the player, the NPCs and the infected all follow, because they share the same animation sequences.

To place the notifies on your own animation, run the `AM_FootstepSounds` animation modifier on the sequence. It adds the left and right notifies for you. See [Use your own animations](../player/use_your_own_animations.md).

For what the AI does with that noise, see [Noise and distractions](..

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
