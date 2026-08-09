# The AI settings asset

Everything you can tune on an AI lives in one Data Asset: its faction, its behavior tree, its senses, what it does when nothing is happening, how it picks a target, its melee ranges, and for a human, the gun it spawns with.

You point a pawn at one of these assets and that is the whole configuration. Change a value in the asset and every AI using it changes.

The assets live in `Content/TheLastTemplate/Blueprints/DataAssets/AI/`.

---

## Three classes, one for each kind of AI

| Class | What it adds | Used by |
|---|---|---|
| `BP_AIDataAsset` | The base. 39 fields every AI needs: faction, tree, senses, idle, targeting, melee. | Nothing directly. It is the parent of the other two. |
| `BP_NPCDataAsset` | The base plus 13 fields only a human uses: weapon, ammo, ranged movement, melee defence level. | `BP_NPCCharacter` |
| `BP_ZombieDataAsset` | The base plus 3 fields only the infected use: `Attack Montage`, `Attack Damage`, `Strike Bone`. | `BP_ZombieCharacter` |

The slot on the pawn is called `Data NPC` on both, and it is typed as the base class, so it accepts a preset of either kind. `BP_ZombieCharacter` carries a second slot, `Zombie Data`.

!!! warning "A zombie dragged into a level is half configured"
    `BP_ZombieCharacter` ships with `Data NPC` set to `DA_Zombie_Roaming` but `Zombie Data` left empty. Fill it in before you play, or make a child class that has it filled. [Add an AI to your level](add_an_ai_to_your_level.md) covers the placement.

---

## The five presets that ship

They are all in `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/`.

| Preset | What it is | What makes it different |
|---|---|---|
| `DA_NPC_Roaming` | Armed human that wanders around where it was placed. | The baseline. `Idle Pattern` is `Roaming`, `Default Weapon Class` is `BP_Weapon_Pistol_01`, `Melee Defense Level` is `Level_01`. |
| `DA_NPC_Melee_Roaming` | The same human with no gun. | `Default Weapon Class` is empty and `Melee Defense Level` is `Level_02`. |
| `DA_NPC_Path` | Human that walks a patrol path. | `Idle Pattern` is `Path` and `Path Id` is `Path_01`. Also the only preset with `Strike Range` at 150 and `Strike Stop Margin` at 20. |
| `DA_NPC_Stand` | Human that holds a position. | `Idle Pattern` is `StandPoint`, `Stand Point Id` is `Stand_01`, and `Move Delay` and `Move Delay Jitter` are both 0 because it does not wander. This is the default on `BP_NPCCharacter`. |
| `DA_Zombie_Roaming` | Infected. | `Behavior Tree` is `BT_Zombie`, `Faction` is `DA_Faction_Infected`, `Idle Overlay` is `Zombie`, `First Attack Delay` drops from 2 to 0.3, `Strike Range` is 100 and `Lose Target Distance` is 3000. |

The four human presets are the same numbers with three or four fields moved. That is the point: most of what makes an enemy feel different is a handful of values, not a new class.

There is no "melee only" switch. An unarmed human is a preset with `Default Weapon Class` cleared.

---

## Make your own preset

1. In `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/`, right click the preset closest to what you want and choose **Duplicate**.
2. Name it, for example `DA_NPC_Sniper`.
3. Open it and change the values you care about.
4. Select the AI in your level, or open your own child class, and set `Data NPC` to your new asset.
5. Save.

To start from nothing instead, right click in the Content Browser, then **Miscellaneous**, then **Data Asset**, and pick `BP_NPCDataAsset` or `BP_ZombieDataAsset` as the class. Duplicating is usually faster, because every field starts at a value that already works.

---

## Identity

| Field | What it does | Shipped |
|---|---|---|
| `Faction` | The side this AI belongs to. Points at a faction Data Asset, which lists who it is hostile to. | `DA_Faction_Hunters` on the humans, `DA_Faction_Infected` on the zombie |
| `Behavior Tree` | The tree the controller runs. | `BT_NPC` on the humans, `BT_Zombie` on the zombie |
| `Idle Overlay` | The animation overlay the character wears outside combat. | `Base` on the humans, `Zombie` on the zombie |

Who attacks who is decided by the faction asset, not by the AI. See [Factions and who fights who](factions_and_targets.md).

---

## Senses and alert

| Field | What it does | Shipped |
|---|---|---|
| `Use Sight` | Whether this AI sees at all. | true |
| `Use Hearing` | Whether this AI hears at all. | true |
| `Lost Sight Grace` | Seconds of lost line of sight before the AI treats the target as lost. | 2 |
| `Search Give Up Time` | Seconds spent searching before it gives up and goes back to idle. | 8 |
| `Alert Turn Time` | Seconds spent turning toward what alerted it. | 1.5 |
| `React To Damage` | Whether being hurt puts it on alert. | true |
| `Damage Investigate Distance` | How far, in centimetres, it will travel to investigate the source of damage. | 900 |

The cones, the ranges and the hearing radius are not in this asset. They are on the perception component. [How the AI sees, hears and reacts](how_the_ai_sees_and_hears.md) covers those, and [Noise, and how to use it](noise_and_distractions.md) covers what makes a sound in the first place.

---

## Idle and roaming

| Field | What it does | Shipped |
|---|---|---|
| `Idle Pattern` | What the AI does with no target: `None`, `Path`, `Roaming` or `StandPoint`. | `Roaming`, except `Path` and `StandPoint` on their presets |
| `Path Id` | Name of the patrol path to follow, when `Idle Pattern` is `Path`. | `Path_01` on `DA_NPC_Path`, empty elsewhere |
| `Stand Point Id` | Name of the stand point to hold, when `Idle Pattern` is `StandPoint`. | `Stand_01` |
| `Roam Zone Id` | Name of the roam zone to stay inside. Empty means it roams around where it stands. | empty on all five |
| `Roam Min Radius` | Closest a roam destination can be picked, in centimetres. | 300 |
| `Roam Max Radius` | Furthest a roam destination can be picked. | 1200 |
| `Roam Separation` | Distance kept between two AI roaming in the same place. | 600 |
| `Move Delay` | Seconds waited at a destination before moving again. | 2, and 0 on `DA_NPC_Stand` |
| `Move Delay Jitter` | Random seconds added to `Move Delay`, so a group does not move in step. | 3, and 0 on `DA_NPC_Stand` |

The three id fields are plain text matched against actors you place in the level, so adding a patrol is placing an actor and typing its name here. [Patrol paths, roam zones and stand points](patrol_paths_and_roam_zones.md) has the actors and the naming.

---

## Choosing a target

| Field | What it does | Shipped |
|---|---|---|
| `Target Stickiness` | How much the target it already has is favoured over a new one, so it does not swap every second. | 400 |
| `Lose Target Distance` | Distance at which the current target is dropped. | 3300, and 3000 on the zombie |
| `Retaliation Window` | Seconds during which whoever last hurt it counts as a fresh attacker. | 5 |
| `Retaliation Bonus` | How strongly that attacker is favoured during the window. | 1500 |

---

## Melee ranges and timing

| Field | What it does | Shipped |
|---|---|---|
| `Melee Enter Range` | Distance at which it switches into melee. | 200 |
| `Melee Exit Range` | Distance at which it drops back out of melee. Larger than the enter range on purpose, so it does not flicker between the two. | 280 |
| `Strike Range` | Distance at which a strike is thrown. | 130, 150 on `DA_NPC_Path`, 100 on the zombie |
| `Strike Stop Margin` | Extra distance kept when stopping to strike. | 15, 20 on `DA_NPC_Path` and the zombie |
| `Melee Cooldown` | Seconds between two attacks. | 1.25 |
| `Combo Min` / `Combo Max` | Fewest and most strikes in one combo. | 2 and 3 |
| `Combo Delay` | Seconds between two strikes inside a combo. | 0.35 |
| `First Attack Delay` | Seconds before the first attack after entering melee. | 2, and 0.3 on the zombie |
| `Face Target On Attack` | Whether it turns to face the target before striking. | true |
| `Face Turn Interp Speed` | How fast that turn is. | 8 |
| `Melee Run Distance` | Beyond this distance it runs to close in instead of walking. | 450 |
| `Run Away Speed Threshold` | Target speed above which it is treated as running away. | 300 |
| `Run Away Dot Threshold` | How far the target must be facing away to count as running away. | -0.35 |
| `Finisher Range` | Distance at which a stealth finisher on this AI is allowed. | 170 |
| `Finisher Facing Dot` | How much the AI must have its back turned for the finisher to be allowed. | -0.7 |

`First Attack Delay` is the field that decides whether an enemy reads as human or as infected. Two seconds gives you time to react. The zombie's 0.3 does not.

[Tune how the AI fights up close](tune_melee_ai.md) goes through these with the animation side attached.

---

## Ranged fighting, humans only

These are on `BP_NPCDataAsset`. A zombie preset does not have them.

| Field | What it does | Shipped |
|---|---|---|
| `Aim Settle Time` | Seconds spent settling the aim before firing. | 0.35 |
| `Strafe Distance` | How far a strafe move goes. | 250 |
| `Strafe Repick Interval` | Seconds before a new strafe destination is picked. | 1.6 |
| `Strafe Repick Jitter` | Random seconds added to that interval. | 0.8 |
| `Ring Strafe Distance` | How far a move goes when circling a close target. | 120 |
| `Ring Repick Interval` | Seconds before a new circling destination is picked. | 1.2 |
| `Ring Repick Jitter` | Random seconds added to that interval. | 0.6 |
| `Ring Snap Distance` | Distance under which it circles instead of strafing. | 250 |
| `Retreat Distance` | How far it backs off when the target is too close. | 700 |
| `Retreat Repath Interval` | Seconds between two retreat paths. | 0.5 |

The two jitter fields exist so that several humans fighting you at once do not move as one block. Set them to 0 and a squad turns into a chorus line.

---

## Weapon, ammo and defence level, humans only

| Field | What it does | Shipped |
|---|---|---|
| `Default Weapon Class` | The weapon the AI spawns holding. Leave it empty for an unarmed enemy. | `BP_Weapon_Pistol_01`, empty on `DA_NPC_Melee_Roaming` |
| `Starting Reserve Ammo` | Rounds carried beyond the loaded magazine. | 12 |
| `Melee Defense Level` | How well this AI defends itself in melee: `Level_01` to `Level_04`. | `Level_01`, and `Level_02` on `DA_NPC_Melee_Roaming` and `DA_NPC_Stand` |

Giving an AI a different gun is this one field. [Give an AI a gun and upgrades](../weapons/give_an_ai_a_gun.md) covers the rest of it.

---

## The infected fields

These three are on `BP_ZombieDataAsset` only.

| Field | What it does | Shipped |
|---|---|---|
| `Attack Montage` | Montage slot for the strike. | empty on `DA_Zombie_Roaming` |
| `Attack Damage` | Damage one strike deals. | 12 |
| `Strike Bone` | The bone name the strike uses. | `spine_02` |

---

## Which class owns a field, and why you should care

If you add a field of your own, put it on the class that needs it. A field on `BP_AIDataAsset` shows up on every preset, human and infected. A field on `BP_NPCDataAsset` shows up only on the human presets.

The same rule explains why a zombie preset has no weapon field. It is not hidden, it does not exist on that class, so nothing in the zombie can read it by accident.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
