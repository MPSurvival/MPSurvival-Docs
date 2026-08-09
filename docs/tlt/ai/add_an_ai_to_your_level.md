# Add an AI to your level

Two AI characters ship with the template: a human and an infected. You put one in your level by dragging it in and picking a Data Asset in the Details panel. There is no controller to assign, no Behavior Tree to hook up, and no graph to open.

- The human: `Content/TheLastTemplate/Blueprints/AI/NPC/BP_NPCCharacter`
- The infected: `Content/TheLastTemplate/Blueprints/AI/Zombie/BP_ZombieCharacter`

The infected is not a child of the human. They are two separate characters built from the same components, so a change on one does not follow to the other.

---

## Place a human enemy

1. Drag `BP_NPCCharacter` from the Content Browser into your level.
2. Select it. In the Details panel, set `Data NPC` to one of the presets in `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/`.
3. Add a `NavMeshBoundsVolume` over the ground the AI should walk on, and scale it to cover the area. Press `P` in the viewport to draw the navigation in green.
4. Play.

That is the whole job. `AI Controller Class` is already `BP_NPCController` and `Auto Possess AI` is already `Placed in World or Spawned`, so the character takes control of itself when the level starts.

If you place the character on ground with no navigation under it, it will stand still forever and never chase anything. That is the single most common reason an AI looks broken.

## Place an infected

Same four steps with `BP_ZombieCharacter`, plus one field.

!!! warning "Set `Zombie Data` on a placed infected"
    `BP_ZombieCharacter` has two Data Asset fields. `Data NPC` ships filled with `DA_Zombie_Roaming`, but `Zombie Data` ships empty. Set it to `DA_Zombie_Roaming` as well. That asset carries the melee fields the infected needs: `Attack Damage`, `Strike Bone` and `Attack Montage`.

---

## The presets that ship

Five presets live in `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/`.

| Preset | What it does when nothing is hostile | Weapon | Faction |
|---|---|---|---|
| `DA_NPC_Roaming` | Wanders around where you placed it | `BP_Weapon_Pistol_01` | `DA_Faction_Hunters` |
| `DA_NPC_Melee_Roaming` | Wanders, fights with its fists | none | `DA_Faction_Hunters` |
| `DA_NPC_Path` | Walks a patrol path called `Path_01` | `BP_Weapon_Pistol_01` | `DA_Faction_Hunters` |
| `DA_NPC_Stand` | Goes to a stand point called `Stand_01` and holds it | `BP_Weapon_Pistol_01` | `DA_Faction_Hunters` |
| `DA_Zombie_Roaming` | Wanders, attacks bare handed | none | `DA_Faction_Infected` |

`DA_NPC_Stand` is what a fresh `BP_NPCCharacter` starts with, so a character dropped in without any editing looks for a stand point named `Stand_01`.

An unarmed enemy is not a mode. It is a preset with `Default Weapon Class` left empty, which is exactly what `DA_NPC_Melee_Roaming` does.

The four NPC presets are the same asset with three or four fields moved. Duplicate the closest one and change those fields rather than starting from `BP_NPCDataAsset`. Every field is listed in [The AI settings asset](ai_settings_data_asset.md).

---

## Which side it fights for

An AI's side is not set on the placed character. It is the `Faction` field inside its Data Asset. Pick a preset and you pick a side.

Three factions ship, and each one lists the other two in its `Hostile Factions`, so everything hostile fights everything else hostile. If your AI ignores you entirely, this is the first thing to check. See [Factions and who fights who](factions_and_targets.md).

---

## What it does when nothing is hostile nearby

The `Idle Pattern` field of the Data Asset picks one of four behaviours.

| `Idle Pattern` | What happens | Field it reads |
|---|---|---|
| `None` | No idle behaviour at all | none |
| `Path` | Walks a list of waypoints in order | `Path Id` |
| `Roaming` | Picks points around itself and walks to them | `Roam Zone Id`, `Roam Min Radius`, `Roam Max Radius` |
| `StandPoint` | Walks to one marker and stays there | `Stand Point Id` |

The three ids are plain text, matched against markers you place in the level. Nothing references anything, so you can add and move markers freely.

| Marker | Field to fill | Use |
|---|---|---|
| `BP_AIPath` | `Path Id`, `Path Index` | One actor per waypoint. Same `Path Id`, ordered by `Path Index` |
| `BP_AIRoamZone` | `Zone Id` | A sphere the AI stays inside while roaming |
| `BP_AIStandPoint` | `Point Id` | A single spot to hold |

All three are in `Content/TheLastTemplate/Blueprints/Environments/AI/`. Full detail in [Patrol paths, roam zones and stand points](patrol_paths_and_roam_zones.md).

An id is only read when `Idle Pattern` matches it. A `Stand Point Id` sitting in a preset set to `Roaming` does nothing.

---

## What the character already carries

You do not add any of these. They are on both characters when you drag one in.

| Component | What it gives you |
|---|---|
| `BP_AIBrain` | Reads the Data Asset, tracks the current target and decides idle, search, ranged or melee |
| `BP_FactionComponent` | Who this character is hostile to |
| `BP_CombatComponent` | Melee attacks, dodges and stealth finishers |
| `BP_VitalsSystem` | Health, and dying |
| `BP_HitReactionManager` | Physical reaction to being hit, and ragdoll |
| `BP_FallDamageComponent` | Damage from falling |
| `BP_NPCWeaponManager` | Holding, firing and reloading a gun. Human only |

## Use your own mesh

Make a child of `BP_NPCCharacter` or `BP_ZombieCharacter`, swap the Skeletal Mesh on the child, and place that instead. Keep the Animation Blueprint. If your mesh uses a different skeleton, you need to retarget the animations, which is the longer job covered in [Add a new enemy type](add_new_ai_type.md).

---

## If nothing happens

Work down this list. One of them is almost always the answer.

- **No navigation.** Press `P`. If there is no green under the AI, it cannot move a step.
- **`Data NPC` is empty.** With no Data Asset there is no Behavior Tree, no faction and no ranges. The AI stands there.
- **`Zombie Data` is empty** on a placed infected.
- **Nobody is hostile.** An AI only reacts to factions listed in its faction's `Hostile Factions`.
- **`Use Sight` or `Use Hearing` is off** in the Data Asset. Both ship on, but a preset you edited may have lost one.
- **`Auto Possess AI` was changed.** It must stay on `Placed in World or Spawned`, or the character never gets a controller.
- **The AI has a gun but no ammunition.** `Starting Reserve Ammo` at `0` gives you an enemy that draws a weapon and never fires.

---

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
