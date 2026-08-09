# Use your own animations

The player, the NPC and the zombie all run the same animation Blueprint on the same skeleton. So an animation you swap once is swapped for all three, and most of the swapping happens in one small class that holds nothing but a list of animations.

Where things are:

- The skeleton: `Content/TheLastTemplate/Meshes/Characters/S_HumanoidCharacter`
- The animation Blueprint: `Content/TheLastTemplate/Blueprints/PlayerCharacter/Animations/ABP_HumanoidCharacter`
- The locomotion layer: same folder, `ABP_LocomotionLayers`, and its child `Childs/ABP_LocomotionLayer_Unarmed`
- Blend spaces, aim offsets, montages and notifies: `Content/TheLastTemplate/Animations/`
- The walk, run, crouch, idle and turn sequences themselves: `Content/TheLastTemplate/Demo/Animations/Characters/`

`SK_PlayerCharacter`, `SK_NPC_01`, `SK_NPC_02`, `SK_Zombie_01` and `SK_Companion` all use `S_HumanoidCharacter`, and its bone names are the Unreal mannequin names. If your own character is already on a mannequin-style skeleton, you have very little work to do.

---

## Which asset owns which animation

| Animation | Where you change it |
|---|---|
| Walk, run and crouch blend spaces, idle, turn in place, stops | `ABP_LocomotionLayer_Unarmed`, in the Details panel |
| Narrow passage and lean | in the graph of `ABP_LocomotionLayers` |
| Aim offsets and the weapon overlay poses | in the graph of `ABP_HumanoidCharacter` |
| Firing, reloading, melee, traversal, throwing, using an item | a field on a Data Asset or on a component |

That last row matters. Nothing that plays as a montage is picked inside the animation Blueprint, so you never open a graph to change a punch or a reload.

---

## Replace a locomotion animation

`ABP_LocomotionLayer_Unarmed` is a child of `ABP_LocomotionLayers`. It has no graph of its own. It only carries the animations that the parent graph plays, which makes it the one asset to open for a locomotion swap.

1. Open `Content/TheLastTemplate/Blueprints/PlayerCharacter/Animations/Childs/ABP_LocomotionLayer_Unarmed`.
2. In the Details panel, find the field for the animation you want to change.
3. Set your own animation or blend space.
4. Compile, then save.

The fields:

| Field | What it feeds |
|---|---|
| `Blendspace Walking` | eight way movement at walking speed |
| `Blendspace Running` | eight way movement at running speed |
| `Blendspace Crouching` | eight way movement while crouched |
| `Idle Animation` | standing still |
| `Crouch Idle Animation` | still while crouched |
| `Normal Turn In Place Animations` | eight turns standing: 45, 90, 135 and 180 degrees, left and right |
| `Crouching Turn In Place Animations` | the same eight turns, crouched |
| `Walking Stop Directions` | the stop that plays when you release the stick while walking |
| `Running Stop Directions` | the same, from a run |
| `Crouching Stop Directions` | the same, crouched |

Each of the three stop fields holds eight animations, not four: forward, backward, left and right, each in a left foot and a right foot version. The layer picks between the two from the foot that is currently planted, and that is reported by the footstep notifies. Fill both versions or the character will slide to a stop on one leg.

If you want a second set of locomotion animations for a weapon family rather than a replacement of this one, make another child of `ABP_LocomotionLayers` instead. See [Weapon overlays and animation layers](weapon_overlays_and_layers.md).
## Montages live on Data Assets

Everything that plays over the locomotion is a montage named on a field, so replacing one is a Details panel edit:

| Montage | Field | On |
|---|---|---|
| Character firing, dry fire, reload, equip | `Character Fire Animation`, `Character Empty Animation`, `Character Reload Animation`, `Character Equip Animation` | the weapon Data Asset |
| The gun's own moving parts | `Weapon Fire Animation`, `Weapon Empty Animation`, `Weapon Reload Animation` | the same asset |
| Melee attacks | `Attack Montages` | `BP_CombatComponent` |
| Dodges | `Dodge Montages` | the same component |
| Stealth finishers, attacker and victim | `Stealth Finishers` | the same component |
| Vault, mantle, hurdle and climb | `Montage` | the traversal Data Asset |
| Throwing | `Throw Montage` | the throwable Data Asset |
| Using an item | `Use Montage` | the item Data Asset |
| The zombie's attack | `Attack Montage` | `BP_ZombieDataAsset` |

The one exception is `AM_Character_Land`, the landing montage, which is named in the graph of each character Blueprint rather than on a Data Asset.

For what a melee montage has to carry beyond the animation itself, see [Add a new melee attack](../melee/add_attack_montage.md).

---

## The notifies you have to keep

Two notifies sit on the ground contact frames of every locomotion animation: `AN_FootstepSound_L` and `AN_FootstepSound_R`, both in `Content/TheLastTemplate/Animations/AnimNotifies/Character/`. They do three separate jobs:

- trace down, read the surface, and play the sound for it
- report a noise event that the AI can hear
- tell the animation Blueprint which foot is down

!!! warning
    An animation with no footstep notifies is silent, invisible to AI hearing, and makes the stop animations plant on the wrong foot. All three failures are quiet. Add the notifies before you judge how a new walk cycle feels.

You do not have to place them by hand. `AM_FootstepSounds`, in `Content/TheLastTemplate/Animations/AnimModifiers/`, is an animation modifier that finds the contact frames and adds the notifies for you.

1. Open your animation sequence.
2. In the Animation Data Modifiers panel, add `AM_FootstepSounds`.
3. Apply it.

Its fields are `Feet Definitions`, a list pairing a bone with a notify track and a notify class, and the two speed limits `Foot Planted Speed Treshold` and `Foot Unlanted Speed Treshold` that decide when a foot counts as down. It ships pointing at `ball_l` and `ball_r`. Change those two bone names if your skeleton calls its toes something else.

Which sound comes out for which surface is a separate question, covered in [Change the sound and the mix](..

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
