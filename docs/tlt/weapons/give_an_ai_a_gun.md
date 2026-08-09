# Give an AI a gun and upgrades

At the end of this page you will have an NPC that spawns with a gun in its hand, opens fire at the distance that gun is meant for, and wears a silencer on the barrel. No HUD, no inventory, no workbench and no parts are involved.

An AI fires the exact same weapon assets as the player. Both `BP_PlayerCharacter` and `BP_NPCCharacter` implement `BPI_WeaponOwner`, so a gun you build for the player already works on an AI. See [How guns work in this template](how_guns_work.md) if that split is new to you.

The two places you will work:

- The AI presets: `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/`
- The NPC: `Content/TheLastTemplate/Blueprints/AI/NPC/BP_NPCCharacter`

---

## The gun is on the preset, not on the pawn

`BP_NPCCharacter` has no weapon field. It has one field that decides everything about this NPC, `Data NPC`, and it ships pointing at `DA_NPC_Stand`. The gun, the reserve and the way the NPC moves in a firefight are all inside that Data Asset.

Four presets ship, and they are the same preset with a few fields moved:

| Preset | `Default Weapon Class` | What it is the example of |
|---|---|---|
| `DA_NPC_Roaming` | `BP_Weapon_Pistol_01` | The baseline, wanders and shoots |
| `DA_NPC_Path` | `BP_Weapon_Pistol_01` | Walks a patrol path |
| `DA_NPC_Stand` | `BP_Weapon_Pistol_01` | Holds a stand point |
| `DA_NPC_Melee_Roaming` | `None` | Unarmed, melee only |

Unarmed is not a mode flag. It is an empty `Default Weapon Class`. That single field is the whole difference between `DA_NPC_Roaming` and `DA_NPC_Melee_Roaming`.

!!! warning
    `Data NPC` accepts any child of `BP_AIDataAsset`, but the firearm fields only exist on `BP_NPCDataAsset`. Point it at a plain `BP_AIDataAsset` child and the NPC spawns unarmed with no warning and no error in the log. Duplicate one of the four `DA_NPC_*` presets and you cannot get this wrong.

---

## Arm an NPC

1. In `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/`, duplicate the preset whose idle behaviour you want. Name it after the role, for example `DA_NPC_Shotgun_Stand`.
2. Open it and set `Default Weapon Class` to a child of `BP_WeaponBase`, for example `BP_Weapon_Shotgun_01`.
3. Set `Starting Reserve Ammo` to the number of spare rounds this NPC gets.
4. Save.
5. Drag `BP_NPCCharacter` into your level and set `Data NPC` to your new preset.
6. Play.

That is the whole job. At spawn, `BP_NPCWeaponManager` creates the weapon actor, attaches it to the socket named in `Weapon Grip Socket_R` on the weapon's Data Asset, fills the magazine to `Clip Size`, and switches the character to the gun's `Weapon Overlay`. You do not add a mesh to the character, you do not pick an animation set, and you do not touch a graph.

The two firearm fields on the NPC Data Asset:

| Field | What it does | Shipped |
|---|---|---|
| `Default Weapon Class` | The weapon actor the NPC spawns holding. Empty means melee only | `BP_Weapon_Pistol_01` on three presets, `None` on the melee one |
| `Starting Reserve Ammo` | Rounds carried outside the magazine | `12` on all four |

---

## Ammo, and what it does not have

The magazine an NPC starts with is `Clip Size` on the gun's Data Asset. The reserve is `Starting Reserve Ammo` on the NPC Data Asset. Those two numbers are the whole fight.

An NPC has no inventory. Ammo boxes are interactables, and only the player interacts, so `Munition Class` on the weapon and the `BP_Interactable_Pickup_Ammo_*` actors never come into it. Nothing refills an NPC reserve during play. If you want an NPC that keeps shooting, give it a large `Starting Reserve Ammo`.

Nothing in `BP_NPCCharacter` drops the gun when the NPC dies, so the player cannot loot it. If you want that, spawn one of the pickup children in `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/Weapons/Childs/` where the body falls.

---

## Drawn or put away

There is no "start holstered" field. The gun goes into the hand at spawn. After that, `BP_AIBrain` decides when the weapon comes out and when it goes away, and moves it between `Weapon Grip Socket_R` and `Unholster Humanoid Socket`, both read from the weapon's Data Asset.

!!! note
    An NPC only ever uses `Unholster Humanoid Socket`. The backpack pair, `Unholster Backpack Socket` and `Use Unholster Backpack Socket`, is read by the player only. A shotgun that rides on the player's backpack will sit on the body socket on an NPC, so give that socket a sensible transform on the NPC skeleton or the gun will hang in the wrong place.

---

## How it fights with that gun

The distance an NPC opens fire at, how many rounds it puts in a burst, how badly it aims and how much it hurts you are all on the **weapon's** Data Asset, in its `AI` group: `AI Engage Range`, `AI Preferred Range`, `AI Burst Count`, `AI Spread Multiplier`, `AI Damage Multiplier` and `AI Damage Multiplier to Player`. Those fields and their shipped values are covered in [Change how a gun feels to shoot](change_how_a_gun_feels.md).

The useful consequence: swapping `Default Weapon Class` changes how the NPC fights without touching the preset. A pistol NPC engages at `2000` and holds at `700`, three rounds at a time. Hand the same NPC `BP_Weapon_Shotgun_01` and it closes to `600`, holds at `350` and fires one shot at a time, because those numbers came with the gun.

What stays on the NPC Data Asset is the movement side of a gunfight. All four presets ship the same values. Distances are in centimetres, times in seconds.

| Field | Shipped | What it sets |
|---|---|---|
| `Aim Settle Time` | `0.35` | How long the NPC settles its aim before it shoots. Raise it to give the player more warning |
| `Strafe Distance` | `250` | How far sideways one strafe move goes |
| `Strafe Repick Interval` | `1.6` | How often a new strafe destination is chosen |
| `Strafe Repick Jitter` | `0.8` | Random spread added to that interval |
| `Ring Strafe Distance` | `120` | The same, for the tighter circling used close in |
| `Ring Repick Interval` | `1.2` | How often the close circle picks a new spot |
| `Ring Repick Jitter` | `0.6` | Random spread added to that interval |
| `Ring Snap Distance` | `250` | The distance at which circling takes over |
| `Retreat Distance` | `700` | How far the NPC backs off when you are too close |
| `Retreat Repath Interval` | `0.5` | How often the retreat path is recomputed |

Every `Repick Interval` has a matching `Jitter` so a group of NPCs does not move on the same beat. Set the jitters to `0` and a squad steps in lockstep, which reads as robotic immediately.

---

## Give that NPC a starting upgrade loadout

`BP_NPCCharacter` carries `BP_WeaponUpgradeComponent`, the same component the player has. On the NPC it ships empty, so a shipped NPC has a stock gun.

1. Open `BP_NPCCharacter`, or better a child of it if only some of your NPCs get upgrades.
2. Select `BP_WeaponUpgradeComponent` in the Components list.
3. Add one row to `Initial Loadouts`.
4. Set `Weapon Class` on that row to the same class you put in `Default Weapon Class`.
5. Add a row to `Upgrades` on that loadout. Set `Upgrade` to an upgrade Data Asset from `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Upgrades/Childs/`, and `Current Level` to which level of it the NPC has, starting at `1` for the first entry in that upgrade's `Levels` list.
6. Repeat step 3 for each weapon class, and step 5 for each upgrade on that weapon.
7. Compile and save.

One row of `Initial Loadouts` is one weapon class, so an NPC that can end up holding either a pistol or a shotgun gets two rows.

An upgrade an NPC does not have is simply left out of the array. `Parts Item` on the component can stay `None`: an NPC never pays for anything.

The upgrade assets themselves are the same ones the player crafts, and writing a new one is on [Add a weapon upgrade](add_a_weapon_upgrade.md).

## Making a squad

Two knobs, two different scopes. Use whichever matches what actually differs.

| What differs between your NPCs | Where you change it |
|---|---|
| The gun, the reserve, the patrol, the movement values | A Data Asset. Duplicate a preset per role and set `Data NPC` per placed actor |
| The starting upgrades | A child class of `BP_NPCCharacter`, because `Initial Loadouts` is on the component |

So five NPCs with five different guns need five Data Assets and no new Blueprint. Five NPCs where one carries a silenced pistol need one extra child class.

---

## Making your own pawn hold a gun

The short path is a child of `BP_NPCCharacter`. It arrives with everything already wired.

If you need a pawn that is not one, it has to carry `BP_NPCWeaponManager` and implement `BPI_WeaponOwner`. That interface is the whole contract between a gun and its holder: `Get Equipped Weapon`, `Get Weapon Mesh`, `Get Aim Origin`, `Get Weapon Ammo`, `Can Fire Weapon`, `Consume Weapon Ammo`, `Consume Reserve Ammo` and `Add Weapon Ammo`. It also needs `BP_WeaponUpgradeComponent` for upgrades to apply, and `BP_AIBrain` with an AI controller for anything to drive it. Childing `BP_NPCCharacter` is much less work for the same result.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
