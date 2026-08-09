# Tune how the AI fights with a gun

An armed human closes to a distance, moves side to side, settles its aim, fires a burst, then does it again. Every number in that sentence is a field in a Details panel. This page tells you which field, and which asset it is on.

Only `BP_NPCCharacter` fights with a gun. The infected has no ranged fields at all.

The AI has to be armed before any of this matters. [Give an AI a gun and upgrades](../weapons/give_an_ai_a_gun.md) covers that part.

---

## Two assets own the ranged behaviour

This is the thing that costs people an evening. The fields are split across two assets, and the split is not where you would guess.

| Asset | What it decides |
|---|---|
| The AI preset, in `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/` | How it **moves**: how far a strafe goes, how often it picks a new spot, how long it settles its aim |
| The weapon Data Asset, in `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Childs/` | How it **shoots**: at what distance, how many rounds at a time, how badly it aims, how much it hurts |

So "this enemy shoots at me from across the map" is a weapon field, not an AI field. The useful side of that: hand the same preset a different gun and it fights differently without you touching the preset.

| Step in a firefight | Field | On |
|---|---|---|
| It walks in until it is allowed to fire | `AI Engage Range` | weapon |
| It moves side to side while it holds you | `Strafe Distance`, `Strafe Repick Interval`, `Strafe Repick Jitter` | AI preset |
| It settles the aim | `Aim Settle Time` | AI preset |
| It fires a burst | `AI Burst Count` | weapon |
| The shots scatter | `AI Spread Multiplier` | weapon |
| A hit lands | `AI Damage Multiplier`, `AI Damage Multiplier to Player` | weapon |
| It gives up on you | `Lost Sight Grace`, `Lose Target Distance` | AI preset |

All distances are in centimetres. All times are in seconds.

---

## The distance it opens fire at

`AI Engage Range` is the distance inside which the AI is allowed to shoot. Outside it, the AI walks in instead.

| Weapon Data Asset | `AI Engage Range` |
|---|---|
| `DA_Pistol_01` and `DA_Pistol_02` | 2000 |
| `DA_Shotgun_01` and `DA_Shotgun_02` | 600 |
| The class default on `BP_WeaponDataAsset` | 1200 |

Twenty metres for a pistol, six for a shotgun. That single field is why swapping `Default Weapon Class` from a pistol to a shotgun turns a careful enemy into one that runs at you, with no other change anywhere.

!!! warning "The firing distance belongs to the gun, not to the enemy"
    Two presets holding `BP_Weapon_Pistol_01` open fire at exactly the same distance, because they read the same weapon Data Asset. To make one careful and one reckless with the same gun, duplicate `DA_Pistol_01`, duplicate the `BP_WeaponBase` child that uses it, set `Weapon Data` on the duplicate to your copy, and put that class in `Default Weapon Class` on the second preset. Editing the original changes every AI carrying that gun, and the player's copy of it.

---

## Moving while it shoots

Three fields on the AI preset. All four human presets ship the same values.

| Field | What it does | Shipped |
|---|---|---|
| `Strafe Distance` | How far one sideways move goes | 250 |
| `Strafe Repick Interval` | Seconds before a new strafe spot is chosen | 1.6 |
| `Strafe Repick Jitter` | Random seconds added to that interval | 0.8 |

A short distance with a short interval reads as twitchy. A long distance with a long interval reads as an enemy that commits to a position and holds it.

The jitter is not decoration. It is what keeps three enemies from moving on the same beat. Set it to 0 and a group turns into a chorus line.

---

## Settling the aim, then firing

| Field | On | What it does | Shipped |
|---|---|---|---|
| `Aim Settle Time` | AI preset | Seconds the AI holds its aim before it shoots | 0.35 |
| `AI Burst Count` | weapon | Rounds fired in one burst | 3 on both pistols, 1 on both shotguns |

`Aim Settle Time` is the player's reaction window. It is the gap between the AI having you and the AI shooting you. Raise it to 1 and the enemy is readable, you can see the aim come up and break line of sight. Drop it to 0.1 and there is nothing to react to.

`AI Burst Count` is not set on the pistols, so they use the 3 on `BP_WeaponDataAsset`. Both shotguns override it to 1, which is what makes a shotgun enemy feel like a shotgun instead of a fast pistol.

---

## How badly it aims and how much it hurts

Three multipliers on the weapon Data Asset. They apply when an AI pulls the trigger, so changing them never touches the player's version of the same gun.

| Field | What it does | Pistols | Shotguns | Class default |
|---|---|---|---|---|
| `AI Spread Multiplier` | Multiplies the weapon's spread while an AI is firing it | 10 | 20 | 1 |
| `AI Damage Multiplier` | Scales the damage an AI's shot does to anything that is not the player | 0.7 | 0.5 | 1 |
| `AI Damage Multiplier to Player` | Scales the damage an AI's shot does to the player | 0.2 on `DA_Pistol_01`, 0.3 on `DA_Pistol_02` | 0.3 | 0.5 |

`AI Spread Multiplier` is the first field to reach for when a firefight feels unfair. Ten times the spread the player gets is what makes an AI miss often enough to be worth fighting. Bring it down to 2 and the same enemy becomes a marksman who kills you from twenty metres. The spread it multiplies is `Ammo Spread` on the same asset, covered in [Change how a gun feels to shoot](../weapons/change_how_a_gun_feels.md).

The two damage multipliers are separate so that AI shooting AI and AI shooting you can be balanced apart. Every shipped gun keeps both below 1, and keeps the one aimed at the player lower than the other. A faction fight you walk into should be lethal between them and survivable for you.

---

## Giving up on a target

| Field | What it does | Shipped |
|---|---|---|
| `Lost Sight Grace` | Seconds of broken line of sight before the target counts as lost | 2 |
| `Lose Target Distance` | Distance at which the target is dropped | 3300 on the humans |

`Lose Target Distance` is deliberately larger than every shipped `AI Engage Range`. A pistol AI at 3000 still has you as a target and is not allowed to shoot, so it closes in. That is the chase.

Set `Lose Target Distance` below `AI Engage Range` and you get an AI that forgets you before it is ever allowed to fire. It looks like broken targeting. It is two numbers in the wrong order.

The rest of losing a target, searching for it and giving up is on [The AI settings asset](ai_settings_data_asset.md).

---

## Two different enemies out of the same parts

Neither of these is a new class and neither is a graph. Both are duplicated Data Assets.

**A careful shooter that keeps its distance.** Duplicate `DA_Pistol_01` and the weapon child that uses it, then on the copy set `AI Engage Range` to 2500, `AI Burst Count` to 2 and `AI Spread Multiplier` to 6. On a duplicated `DA_NPC_Stand`, set `Aim Settle Time` to 0.8, `Strafe Distance` to 400 and `Strafe Repick Interval` to 2.5. It holds a position, takes its time, and punishes you for standing still.

**Something that runs at you.** Set `Default Weapon Class` to `BP_Weapon_Shotgun_01`, which already engages at 600 and fires one shot at a time. On a duplicated `DA_NPC_Roaming`, set `Aim Settle Time` to 0.15, `Strafe Distance` to 150 and `Strafe Repick Interval` to 0.8. Raise `Melee Enter Range` above 200 and it stops shooting and comes at you with its hands sooner, which is on [Tune how the AI fights up close](tune_melee_ai.md).

---

## Fields you will see that nothing reads yet

Said plainly so you do not spend an evening tuning them and wondering why nothing changes.

- **`AI Preferred Range`**, on the weapon Data Asset. It ships at 700 by default and 350 on both shotguns, and nothing in the project reads it. The distance an AI actually holds is decided by `AI Engage Range`.
- **`Ring Strafe Distance`, `Ring Repick Interval`, `Ring Repick Jitter` and `Ring Snap Distance`**, on the AI preset. They belong to the close circling task, `BTT_MeleeCircle`, and that task does not circle.
- **`Retreat Distance` and `Retreat Repath Interval`**, on the AI preset. `BTT_MeleeRetreat` is empty, so an armed AI does not back away when you close the distance.

An enemy that should hold you at range has to be built from `AI Engage Range` and `Melee Enter Range`, not from those.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
