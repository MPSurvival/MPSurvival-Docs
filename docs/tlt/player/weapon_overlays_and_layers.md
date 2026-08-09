# Weapon overlays and animation layers

Two separate things decide how a character holds a weapon and how it moves while holding it.

An **overlay** is one pose blended onto the upper body. It is what makes the character look like it is carrying a shotgun instead of nothing. Which pose is used comes from an enum, `E_OverlayLayers`, and a weapon picks its own value in its Data Asset.

An **animation layer** is the locomotion set itself: the idle, the walk and run blend spaces, the stops, the turns in place. Layers live in their own animation blueprint that gets linked into the main one, so a weapon family can have its own walk cycles without you opening the main graph.

The assets involved:

| Asset | Path |
|---|---|
| `E_OverlayLayers` | `Content/TheLastTemplate/Blueprints/Enumerations/Locomotion/` |
| `ABP_HumanoidCharacter` | `Content/TheLastTemplate/Blueprints/PlayerCharacter/Animations/` |
| `ALI_LocomotionLayers` | same folder |
| `ABP_LocomotionLayers` | same folder |
| `ABP_LocomotionLayer_Unarmed` | same folder, under `Childs/` |
| Overlay poses | `Content/TheLastTemplate/Animations/AnimSequences/Characters/Overlay/` |
| Aim offsets | `Content/TheLastTemplate/Animations/AimOffsets/` |

`BP_PlayerCharacter`, `BP_NPCCharacter` and `BP_ZombieCharacter` all run `ABP_HumanoidCharacter`, and all three link `ABP_LocomotionLayer_Unarmed`. What you change here you change for the player, the NPCs and the zombies at once.

An overlay does not move the feet, does not change speed, and does not decide what the weapon does when you pull the trigger. It is upper body posture only. Movement speed lives in [Change movement speeds and gaits](movement_speeds_and_gaits.md).

---

## The six overlays that ship

| Value | What it is for | Poses that ship for it |
|---|---|---|
| `Base` | Empty hands. It is also the value all four NPC Data Assets use. | `AS_Humanoid_Overlay_UnarmedAim` |
| `Shotgun` | The two shotguns, `DA_Shotgun_01` and `DA_Shotgun_02` | `_Shotgun_Hip`, `_Shotgun_ADS`, `_Shotgun_Run`, `_Shotgun_Crouch` |
| `Pistol` | The two pistols, `DA_Pistol_01` and `DA_Pistol_02` | `_Pistol_Hip`, `_Pistol_ADS`, `_Pistol_Run` |
| `Throw` | Holding a throwable | `AS_Humanoid_Overlay_Throw` |
| `Combat` | Unarmed melee | `AS_Humanoid_Overlay_Combat` |
| `Zombie` | The zombie. `DA_Zombie_Roaming` sets it. | `AS_Humanoid_Overlay_Zombie` |

The pose files all start with `AS_Humanoid_Overlay_`. `Pistol` and `Shotgun` each have their own subfolder because they carry several poses. The shotgun has a crouch pose and the pistol does not.

---

## Point a weapon at an overlay

This is the whole job for a weapon that fits one of the six values above. No graph.

1. Open the weapon's Data Asset, for example `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Childs/DA_Pistol_01`.
2. Set `Weapon Overlay` to the value you want.
3. Save, then play and equip the weapon.

For an AI, the same idea lives on the AI Data Asset: `Idle Overlay` in `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/`. `DA_NPC_Path`, `DA_NPC_Stand`, `DA_NPC_Roaming` and `DA_NPC_Melee_Roaming` all ship on `Base`, and `DA_Zombie_Roaming` ships on `Zombie`.

---

## Change what a weapon family looks like

If you want a shotgun held differently, you do not need a new overlay value. Replace the poses.

1. Open `AS_Humanoid_Overlay_Shotgun_Hip`. That is the pose the shotgun is carried in.
2. Either re-import over it with your own single frame animation, or make your own sequence and point the matching node in `ABP_HumanoidCharacter` at it.
3. Do the same for `_ADS`, `_Run` and `_Crouch` so the four states agree with each other.

The four poses are separate on purpose: a weapon that reads well at the hip usually reads badly while running.

## Left hand IK

The left hand is placed by a two bone IK driven from four fields on the weapon Data Asset. There is no graph work.

| Field | What it does |
|---|---|
| `Use Custom Left IK` | Turns the left hand IK on for this weapon |
| `Use Custom Left IKSprinting` | Whether that IK stays on while sprinting |
| `Join Target Location` | The elbow target. It is only read when `Use Custom Left IK` is on. |
| `Weapon Grip Socket R` | The socket on the character skeleton the weapon is attached to |

Shipped values:

| Data Asset | `Join Target Location` | `Weapon Grip Socket R` |
|---|---|---|
| `DA_Pistol_01`, `DA_Pistol_02` | `300, 500, -5000` | `PistolHandRSocket` |
| `DA_Shotgun_01`, `DA_Shotgun_02` | `100, -100, 0` | `ShotgunHandRSocket` |

All four ship with `Use Custom Left IK` on and `Use Custom Left IKSprinting` off.

Both weapon skeletons that ship, `S_Pistol` and `S_Shotgun`, carry a socket called `LeftHandIK_Aiming`. Give your own weapon skeleton a socket with that exact name and put it where the left hand should sit. The two shipped `Join Target Location` values are far apart, so copy the one for the weapon closest to yours and move it from there.

The character skeleton also carries `InsertMunitionPistolSocket` and `InsertMunitionShotgunSocket`, named on the weapon Data Asset by `Left Hand Magazine Socket`. That one is the reload, not the carry pose.

---

## The layer interface and the one class that ships

`ALI_LocomotionLayers` declares five layers. `ABP_LocomotionLayers` implements all five, and `ABP_LocomotionLayer_Unarmed` is the only child that ships.

| Layer | Covers |
|---|---|
| `LocomotionLayer_Idle` | Standing and crouched idle, plus the turns in place |
| `LocomotionLayer_Movement` | Walking, running and crouched movement |
| `LocomotionLayer_Stop` | The foot planted stops at the end of a move |
| `LocomotionLayer_NarrowPassage` | The shuffle used in tight spaces |
| `LocomotionLayer_Falling` | In air |

A child class fills in animations, not graphs. These are the variables it sets:

| Variable | Shipped in `ABP_LocomotionLayer_Unarmed` |
|---|---|
| `Idle Animation` | `AS_Humanoid_Stand_Idle_Loop` |
| `Crouch Idle Animation` | `AS_Humanoid_Crouch_Idle_Loop` |
| `Blendspace Walking` | `BS_HumanoidWalking` |
| `Blendspace Running` | `BS_HumanoidRunning` |
| `Blendspace Crouching` | `BS_HumanoidCrouching` |
| `Walking Stop Directions` | Eight stop sequences, four directions times the planted foot |
| `Running Stop Directions` | The same eight walk stop sequences again |
| `Crouching Stop Directions` | Eight stop sequences, four directions times the planted foot |
| `Normal Turn In Place Animations` | The standing turn set |
| `Crouching Turn In Place Animations` | The crouched turn set |

---

## Make your own layer class

Do this when a weapon family needs its own walk cycles, not just a different upper body pose.

1. Right click `ABP_LocomotionLayers`, then **Create Child Blueprint Class**. Name it after the family, for example `ABP_LocomotionLayer_Rifle`.
2. Put it next to the one that ships, in `.../Animations/Childs/`.
3. In **Class Defaults**, fill the variables from the table above with your own blend spaces and sequences.
4. Compile and save.
5. In the character that should use it, change the class passed to the **Link Anim Class Layers** node.

Step 5 is the part that is not data. `BP_PlayerCharacter`, `BP_NPCCharacter` and `BP_ZombieCharacter` each call **Link Anim Class Layers** once with `ABP_LocomotionLayer_Unarmed`, and nothing swaps it afterwards. If you want the layer to change when a weapon is equipped, that is where you call it again.

---

## Adding a seventh overlay

The six values are not open ended. A new one costs two edits:

1. Add an entry to `E_OverlayLayers`.
2. Handle that entry in `ABP_HumanoidCharacter`, next to the six that are already there, and give it a pose.

There is no Data Asset that adds an overlay for you. If your weapon can live on `Pistol` or `Shotgun` with different poses, take that route instead: it is a lot cheaper.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
