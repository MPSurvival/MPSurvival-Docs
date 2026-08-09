# Add a new enemy type

You end up with a third enemy that is yours: your own class, your own mesh, your own numbers, sitting next to the human and the infected that ship. You build it from parts that are already there. There is no new controller to write, no new Behavior Tree to build, and no graph to open.

Have a skeletal mesh ready before you start. If it uses the same skeleton as the characters that ship, the animation side is already done.

---

## Pick which one to start from

The two AI characters are siblings, not parent and child, so you choose one and copy nothing from the other.

| Start from | Take this one when | What it brings |
|---|---|---|
| `BP_NPCCharacter` | Your enemy carries a gun, or might later | Everything below, plus `BP_NPCWeaponManager` and `BP_WeaponUpgradeComponent` |
| `BP_ZombieCharacter` | Your enemy only ever fights bare handed | The same base, minus the two weapon components, plus a second Data Asset slot for the strike |

Both are in `Content/TheLastTemplate/Blueprints/AI/`.

A child of either one already carries the brain, the faction component, the melee component, health, hit reactions and fall damage. They are listed in [Add an AI to your level](add_an_ai_to_your_level.md). You do not add any of them, and you should not remove any of them.

An unarmed human is not a reason to start from the infected. Clearing `Default Weapon Class` on the preset is enough, which is exactly what `DA_NPC_Melee_Roaming` does.

---

## Build the class

1. In the Content Browser, right click `BP_NPCCharacter` or `BP_ZombieCharacter` and choose **Create Child Blueprint Class**. Name it, for example `BP_NPC_Brute`.
2. Open it and select the `Mesh` component. Set `Skeletal Mesh Asset` to your mesh.
3. Leave `Anim Class` on `ABP_HumanoidCharacter`. The player, the human and the infected all run that one animation Blueprint, so a mesh on the same skeleton needs nothing else. A mesh on a different skeleton has to be retargeted first, which is covered in [Use your own animations](../player/use_your_own_animations.md).
4. Compile and save.

That is the class. Everything that makes it feel like a different enemy is in the next two steps.

---

## Give it its own settings asset

Your child class still points at a preset that other AI share. Editing that preset changes them too, so make your own.

1. In `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/`, duplicate the preset closest to what you want. Name it, for example `DA_NPC_Brute`.
2. Back in your child class, set the class default `Data NPC` to your new asset.
3. Change the values you care about in the asset. Every field is listed in [The AI settings asset](ai_settings_data_asset.md).

Keep the pairing that ships. A child of `BP_NPCCharacter` takes a `BP_NPCDataAsset` preset, whose `Behavior Tree` is `BT_NPC`. A child of `BP_ZombieCharacter` takes a `BP_ZombieDataAsset` preset, whose `Behavior Tree` is `BT_Zombie`. The `Data NPC` slot is typed as the shared base class, so it will happily accept the wrong kind of preset without complaining.

!!! warning "Fill `Zombie Data` on the class, not on each placed enemy"
    A child of `BP_ZombieCharacter` inherits `Zombie Data` empty, and an empty `Zombie Data` means no `Attack Damage` and no `Strike Bone`. Set it once on the class default of your child and every copy you place is configured. This is the main reason to make a child class instead of editing placed actors.

---

## Where the rest of the numbers live

Not everything is in the Data Asset. This is the map.

| What you want to change | Where it is |
|---|---|
| Faction, senses, idle pattern, target choice, melee ranges and timing, weapon and ammo | The Data Asset in `Data NPC`. See [The AI settings asset](ai_settings_data_asset.md) |
| Damage of one strike, reach, knockback, hit stop, dodge chance, and the `Attack Montages` list | `BP_CombatComponent` on your class. See [Tune how the AI fights up close](tune_melee_ai.md) |
| Health | The `Vitals` list on `BP_VitalsSystem`. See [Change health and stamina, or add a new vital](../damage/health_stamina_and_new_vitals.md) |
| How fast it walks and runs | `Locomotion Gaits Settings` on the class, a list of gaits with their movement settings. `Running` ships with `Max Walk Speed` at 400. See [Movement speeds and gaits](../player/movement_speeds_and_gaits.md) |
| How it reacts to being shot, and how it ragdolls | `BP_HitReactionManager`. See [Tune hit reactions](../damage/tune_hit_reactions.md) |
| Damage from falling | `BP_FallDamageComponent` |
| The gun it spawns with | `Default Weapon Class` on the Data Asset. See [Give an AI a gun and upgrades](../weapons/give_an_ai_a_gun.md) |

The two shipped melee attacks are `AM_Combat_Attack_01` and `AM_Combat_Attack_02`, in `Content/TheLastTemplate/Animations/Montages/Characters/Combat/`. `BP_CombatComponent` holds no animation of its own, so the montage list is filled per character. A child inherits the list of the character it came from. Writing a new attack is covered in [Add a new melee attack](../melee/add_attack_montage.md).

---

## What actually separates the two enemies that ship

This is worth reading once, because it tells you how much you really have to change.

The human and the infected run the same brain, the same faction component, the same melee component, the same health and the same hit reactions. Every idle number, every targeting number and almost every melee number is identical between them. What differs:

| Field | Human | Infected |
|---|---|---|
| `Faction` | `DA_Faction_Hunters` | `DA_Faction_Infected` |
| `Behavior Tree` | `BT_NPC` | `BT_Zombie` |
| `Idle Overlay` | `Base` | `Zombie` |
| `First Attack Delay` | 2 | 0.3 |
| `Strike Range` | 130 | 100 |
| `Strike Stop Margin` | 15 | 20 |
| `Lose Target Distance` | 3300 | 3000 |
| `Move Delay` / `Move Delay Jitter` | 2 and 3 | 3 and 2 |
| Weapon fields | pistol and 12 spare rounds | the class has no weapon fields at all |

`First Attack Delay` is the one that does the most work. Two seconds gives you time to back off. Three tenths does not.

So a new enemy type is usually a mesh, a faction, an overlay, and four or five numbers. Reach for a new class when you need a different mesh or a different set of montages, not when you only need different numbers.

---

## Place it and check it

1. Drag your class into the level.
2. Confirm `AI Controller Class` still holds the controller it inherited, and that `Auto Possess AI` is still `Placed in World or Spawned`.
3. Put a `NavMeshBoundsVolume` over the ground and press `P` to see the green navigation.
4. Play, and walk into its line of sight.

Build and judge your enemy against the player. Two AI on hostile factions will pick each other as targets, but melee between two AI is not something the template runs, so watching two of your new enemies swing at each other tells you nothing.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
