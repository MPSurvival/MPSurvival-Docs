# How the player character works

The player is one Blueprint: `Content/TheLastTemplate/Blueprints/PlayerCharacter/BP_PlayerCharacter`. It is a plain Unreal `Character`. Everything it can do is an actor component sitting on it, and each component owns one system and nothing else.

This page says which component owns what, what drives the animation, and which page to open when you want to change something.

---

## What is on the player character

Twelve components carry the gameplay. To change what a system does, open the component, not the character.

| Component | What it owns | Where it is covered |
|---|---|---|
| `BP_VitalsSystem` | Health, stamina, dying | [Health, stamina and new vitals](../damage/health_stamina_and_new_vitals.md) |
| `BP_CombatComponent` | Melee attacks, dodges, stealth finishers | [How melee works](../melee/melee_how_it_works.md) |
| `BP_HitReactionManager` | Physical hit reactions and the ragdoll handover | [Tune hit reactions](../damage/tune_hit_reactions.md) |
| `BP_FallDamageComponent` | Damage from a fall | [Fall damage and death](../damage/fall_damage_and_death.md) |
| `BP_FactionComponent` | Who counts as an enemy | [Factions and targets](../ai/factions_and_targets.md) |
| `BP_PlayerWeaponManager` | Guns carried, ammo, the weapon wheel | [How guns work](../weapons/how_guns_work.md) |
| `BP_WeaponUpgradeComponent` | Attachments and weapon upgrades | [Add a weapon upgrade](../weapons/add_a_weapon_upgrade.md) |
| `BP_PlayerThrowableManager` | Throwables and the throw arc | [How throwables work](../throwables/how_throwables_work.md) |
| `BP_PlayerInventoryManager` | Backpack, crafting, collectibles | [Add an item](../inventory/add_an_item.md) |
| `BP_PlayerSkillManager` | Skills and what they buff | [Add or change a skill](../progression/add_or_change_a_skill.md) |
| `BP_PlayerInspectManager` | Holding an object up and turning it | [How inspection works](../progression/how_inspection_works.md) |
| `BP_PlayerPhotoModeManager` | Photo mode | [Photo mode](../extras/photo_mode_overview.md) |

The rest of the components are the rig, not gameplay:

- `boomTPS` and `cameraTPS`, the camera you play on
- `boomDeath` and `cameraDeath`, the camera that takes over when you die
- `lightSocket` and `BP_SocketLightChild`, where the flashlight hangs
- `BP_MotionWarping`, which lets traversal montages land your hands on the real edge

!!! warning "Six camera fields on the character are not the knobs they look like"
    `Spring Arm Value`, `Spring Arm Value Crouch`, `Field Of View Camera`, `Capsule Radius Value`, `Socket Offset` and `Socket Offset Crouch` all read `0` on `BP_PlayerCharacter`. They are filled in at Begin Play from the real `boomTPS`, `cameraTPS` and capsule components. Typing a value into them changes nothing. Set it on the component instead.

---

## The one animation blueprint

One animation Blueprint drives the player, the NPCs and the zombies. Four assets are involved.

| Asset | What it is |
|---|---|
| `Blueprints/PlayerCharacter/Animations/ABP_HumanoidCharacter` | The animation Blueprint on the mesh. It works out speed, angle, gait, aim, overlay and IK, and owns the aim offsets `AO_HumanoidAO` and `AO_WeaponAO` |
| `Blueprints/PlayerCharacter/Animations/ALI_LocomotionLayers` | The animation layer interface. It is the list of layers `ABP_HumanoidCharacter` links in |
| `Blueprints/PlayerCharacter/Animations/ABP_LocomotionLayers` | The base layer class. It holds the state machines, plus the lean and narrow passage blend spaces |
| `Blueprints/PlayerCharacter/Animations/Childs/ABP_LocomotionLayer_Unarmed` | The one child layer that ships. It holds no graph, only the animation assets |

The split is the point. `ABP_HumanoidCharacter` decides what the character is doing. The layer decides which animation plays. So swapping animations means filling in a child layer class, not editing a graph.

---

## How the character talks to the animation blueprint

Through `Blueprints/Interfaces/BPI_LocomotionGaitInterface`, in both directions. Any character that implements the interface can use `ABP_HumanoidCharacter`.

- The character answers `Get Gait Data`. The animation Blueprint asks for the current gait and its settings.
- The character calls `Receive Aim State`, `Receive Overlay State`, `Receive Locomotion Extras`, `Receive Doing Traversal Action`, `Receive Use Overlay`, `Receive Traversal IK Left` and `Receive Traversal IK Right` on the animation Blueprint when those things change.

`BP_NPCCharacter` and `BP_ZombieCharacter` answer the same interface, which is why they share the animation Blueprint with the player.

---

## Turn in place

When you move the mouse standing still, the capsule turns at once. The mesh does not follow it. `Root Yaw Offset` holds the body back so it keeps facing where it was.

Once the offset is wide enough, the layer picks a turn animation and plays it, and that animation drains the offset back to zero through its own curves. Eight animations per stance, at 45, 90, 135 and 180 degrees, left and right:

- standing, in `Normal Turn In Place Animations`, from `Demo/Animations/Characters/TurnInPlace/Normal/`
- crouched, in `Crouching Turn In Place Animations`, from `Demo/Animations/Characters/TurnInPlace/Crouch/`

Both are fields on `ABP_LocomotionLayer_Unarmed`. `Root Yaw Offset Mode` on `ABP_HumanoidCharacter` ships as `Accumulate`, and the other two values, `BlendOut` and `Hold`, exist for the moments where the body must stop drifting.
## Lean, direction and play rate

`BS_HumanoidLean` in `Content/TheLastTemplate/Animations/BlendSpaces/` bends the body into a turn while running. It is driven by `Lean Angle`, and it is built from five poses in `Demo/Animations/Characters/Lean/`: base, front, back, left and right.

Direction is four values, not eight. `E_LocomotionDirections` is `Forward`, `Backward`, `Right` and `Left`, and the blend spaces cover everything between them. What comes in eight is the stop: `Walking Stop Directions`, `Running Stop Directions` and `Crouching Stop Directions` each hold four directions times which foot is planted, so the character stops on the foot that is down.

The blend spaces that ship:

| Blend space | Where it is set |
|---|---|
| `BS_HumanoidWalking` | `Blendspace Walking` on the child layer |
| `BS_HumanoidRunning` | `Blendspace Running` on the child layer |
| `BS_HumanoidCrouching` | `Blendspace Crouching` on the child layer |
| `BS_HumanoidLean` | inside `ABP_LocomotionLayers` |
| `BS_HumanoidNarrowPassage` | inside `ABP_LocomotionLayers` |

---

## Footsteps

`Animations/AnimNotifies/Character/AN_FootstepSound` is the parent notify. It traces down from the foot, reads the physical material it lands on and plays the matching cue from `Content/TheLastTemplate/Audios/Footsteps/`. Six surfaces ship: concrete, grass, ground, metal, water and wood. The gait sets how loud it is.

`AN_FootstepSound_L` and `AN_FootstepSound_R` are its two children, one per foot. You do not place them by hand: `Animations/AnimModifiers/AM_FootstepSounds` is an Animation Modifier that adds them to a sequence for you, off the `ball_l` and `ball_r` bones.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
