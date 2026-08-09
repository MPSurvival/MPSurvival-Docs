# Change how a gun feels to shoot

Everything that makes a gun feel light or heavy lives in one Data Asset. You do not open a graph to change any of it.

The four that ship are in `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Childs/`:

| Data Asset | `Weapon Display Name` | Shape |
|---|---|---|
| `DA_Pistol_01` | Semi-Auto Pistol | One shot per click, 16 rounds |
| `DA_Pistol_02` | Revolver | Same body, `Automatic Fire` turned on |
| `DA_Shotgun_01` | Pump Shotgun | 8 pellets, 8 shells, wide |
| `DA_Shotgun_02` | Short Shotgun | Same, tighter start, shorter range |

Open one, change a number, press play. The values below are the shipped ones, so you always have two reference points to aim between.

---

## Rate of fire

| Field | What it does | Pistols | Shotguns |
|---|---|---|---|
| `Fire Rate` | Seconds between two shots | `0.2` | `0.75` |
| `Automatic Fire` | Keep firing while the button is held | off, on for `DA_Pistol_02` | off |
| `Automatic Fire Rate` | Seconds between shots while held | `0` / `0.05` | `0` |
| `Clip Size` | Rounds before a reload | `16` | `8` |

`Automatic Fire Rate` is only read when `Automatic Fire` is on, so leaving it at `0` on a semi-automatic gun is correct and changes nothing.

---

## Pellets and spread

`Amount Bullet Per Shoot` is how many traces one shot sends. It is `1` on both pistols and `8` on both shotguns. That single field is the whole difference between a rifle and a scattergun.

| Field | What it does | Pistols | `DA_Shotgun_01` | `DA_Shotgun_02` |
|---|---|---|---|---|
| `Ammo Spread` | How wide a shot starts | `50` | `200` | `100` |
| `Max Ammo Spread` | The widest it can get while you keep firing | `500` | `1000` | `1000` |
| `Spread Recovery Speed` | How fast it settles back once you stop | `5` | `5` | `5` |

These are not degrees. Treat them as a scale of their own and calibrate against the four shipped guns: `50` is a tight pistol, `200` is a wide shotgun.

---

## Recoil and camera shake

| Field | What it does | Pistols | Shotguns |
|---|---|---|---|
| `Recoil Pitch Amount` | How far the camera kicks up | `10` | `20` |
| `Recoil Yaw Amount` | How far it kicks sideways | `2` | `5` |
| `Camera Shake` | The shake class played on each shot | `CS_ShootCamera` | `CS_ShootCamera_Heavy` |
| `Force Camera Shake` | How strongly that shake is played | `3` / `2` | `8.5` |

Both shake classes are in `Content/TheLastTemplate/Blueprints/Misc/CameraShakes/`. `CS_ShootCamera` is the short one, `CS_ShootCamera_Heavy` the long, wide one. To take the shake off one gun without touching the class, set `Force Camera Shake` to `0`.

There is no recoil recovery field. The Data Asset only exposes a recovery speed for spread, so if you want a different return, that is the one lever the panel gives you.

---

## Aiming

| Field | What it does | Pistols | Shotguns |
|---|---|---|---|
| `Camera FOV` | Field of view while the gun is out | `90` | `90` |
| `Zoom Value` | Field of view while aiming, so aiming narrows from one to the other | `50` | `50` |
| `Aim Fire Delay` | Seconds after you raise the gun before it will fire | `0.1` | `0.3` |
| `Crosshair Display Delay After Aim` | Seconds before the crosshair appears | `0.2` | `0.2` |
| `Aim Movement Speed` | Movement speed multiplier while aiming | `0.65` | `0.65` |
| `Spring Arm Right` | Shoulder offset of the camera for this gun | `0, 40, 15` | `0, 40, 15` |
| `Spring Arm Right Crouching` | Same offset while crouched | `0, 40, -10` | `0, 40, -10` |
| `Enable Aim Immersive Camera` | Extra camera movement while aiming | on | on |
| `Aim Camera Intensity` | How much of it | `1.5` | `2` |

`Aim Fire Delay` is the cheapest way to make a gun feel heavy. The shotguns take three times as long as the pistols to be ready, and that alone reads as weight before any recoil is felt.

---

## Range and damage

| Field | What it does | Pistols | Shotguns |
|---|---|---|---|
| `Bullet Max Distance` | How far a shot reaches, in centimetres | `5000` | `5000` / `4000` for `DA_Shotgun_02` |
| `Base Damage` | Damage carried by one hit | `30` | `40` |

`5000` is fifty metres. Past that distance the shot simply does not reach.

Because a shotgun sends eight traces, how many of them land decides how much a shot takes off. Where the hit lands matters too: the bone multipliers are not on the weapon, they are described in [How damage works](../damage/how_damage_works.md).

!!! warning
    Workbench upgrades recompute fire rate, spread, recoil, magazine size, range and damage from the Data Asset at runtime. A gun you tuned to feel right unupgraded will not feel the same once the player has bought two levels of something. Test both. See [Add a weapon upgrade](add_a_weapon_upgrade.md).

---

## Sound, muzzle flash and montages

| Field | Pistols | Shotguns |
|---|---|---|
| `Weapon Fire Sound` | `SC_Pistol_Fire` | `SC_Shotgun_Fire` |
| `Empty Sound` | `SC_DryFire` | `SC_DryFire` |
| `Magazine Drop Sound` | empty | empty |
| `Muzzle Flash Effect` | `NS_Muzzle_Pistol` | `NS_Muzzle_Shotgun` |
| `Muzzle Flash Scale` | `2, 2, 2` | `3, 3, 3` |

`Magazine Drop Sound` ships empty on all four guns. Fill it if you want the magazine to be heard hitting the floor.

There are two sets of montages. The `Character ...` ones play on the player or the AI holding the gun. The `Weapon ...` ones play on the gun's own skeletal mesh, for a slide or a bolt.

| Field | Pistols | Shotguns |
|---|---|---|
| `Character Fire Animation` | `AM_Character_Pistol_Fire` | `AM_Character_Shotgun_Fire` |
| `Character Empty Animation` | `AM_Character_Pistol_EmptyFire` | `AM_Character_Shotgun_EmptyFire` |
| `Character Reload Animation` | `AM_Character_Pistol_Reload` | `AM_Character_Shotgun_Reload` |
| `Character Equip Animation` | empty | empty |
| `Weapon Fire Animation` | `AM_Pistol_Fire` | empty |
| `Weapon Empty Animation` | empty | empty |
| `Weapon Reload Animation` | empty | empty |

Only the pistols use a weapon montage, for the slide. Every empty field above is a field you may leave empty: a gun whose mesh has no moving part needs none of them.

`Weapon Overlay` picks the upper body pose the character holds. `Pistol` on the two pistols, `Shotgun` on the two shotguns. It changes how the character stands and walks with the gun, so it is part of the feel even though it is not a number.

---

## Shells and magazines

| Field | What it does | Pistols | Shotguns |
|---|---|---|---|
| `Ejected Shell` | The actor spawned per shot | `BP_Eject_Ammo_Pistol_01` | `BP_Eject_Ammo_Shotgun_01` |
| `Ejection Force` | How hard it is thrown | `300` | `130` |
| `Shell Ejection Rotation Force` | How hard it spins | `60, 5000, 500` | `65, -50, 500` |
| `Eject Magazine Class` | The actor dropped on reload | `BP_Eject_Magazine_Pistol_01` | empty |
| `Magazine Ejection Force` | How hard it is dropped | `250` | `0` |
| `Magazine Ejection Rotation Force` | How hard it spins | `65, -50, 500` | `0, 0, 0` |
| `Ejected Magazine Socket` | Socket the magazine leaves from | `MagazineEject` | empty |
| `Left Hand Magazine Socket` | Socket the fresh magazine is held in | `InsertMunitionPistolSocket` | `InsertMunitionShotgunSocket` |

The shotguns leave the magazine fields empty on purpose: a pump shotgun has nothing to drop. If you leave `Eject Magazine Class` empty, nothing spawns and the other magazine fields are ignored.

The classes themselves are in `Content/TheLastTemplate/Blueprints/Weapons/Ejects/`. A shell is a small actor with one mesh, so making a bigger brass for a rifle means duplicating `BP_Eject_Ammo_Pistol_01` and swapping the mesh.

---

## The fields that only matter when an AI holds the gun

The same Data Asset is read whether the player or an NPC carries the gun. These seven fields are ignored on the player.

| Field | `DA_Pistol_01` | `DA_Shotgun_01` |
|---|---|---|
| `AI Spread Multiplier` | `10` | `20` |
| `AI Engage Range` | `2000` | `600` |
| `AI Preferred Range` | `700` | `350` |
| `AI Burst Count` | `3` | `1` |
| `AI Damage Multiplier` | `0.7` | `0.5` |
| `AI Damage Multiplier to Player` | `0.2` | `0.3` |

An armed NPC is deliberately much less accurate than you: the spread is multiplied by ten or twenty, and the damage it does to you is cut to a fifth or a third. Raise `AI Damage Multiplier to Player` and a shotgun NPC becomes lethal very quickly. The rest of the ranged AI behaviour is on [Give an AI a gun and upgrades](give_an_ai_a_gun.md).

---

## Three feels, from what already ships

| You want | Duplicate | Then move |
|---|---|---|
| A light sidearm | `DA_Pistol_01` | `Fire Rate` and `Aim Fire Delay` down, `Force Camera Shake` down, `Ammo Spread` down |
| A heavy shotgun | `DA_Shotgun_01` | `Amount Bullet Per Shoot` and `Ammo Spread` up, `Fire Rate` up, `Force Camera Shake` up, `Camera Shake` on `CS_ShootCamera_Heavy` |
| A fast automatic | `DA_Pistol_02` | `Automatic Fire` stays on, `Automatic Fire Rate` down for a faster gun, `Max Ammo Spread` up so long bursts open out, `Recoil Pitch Amount` down so the climb stays readable |

Change one field at a time and play. Recoil, shake and spread all pull in the same direction, and moving three of them at once usually overshoots.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
