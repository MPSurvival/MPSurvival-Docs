# Add or change a player upgrade

An upgrade is a permanent bonus the player buys with supplements, on the **Skills** page of the backpack. Seven of them ship. Adding one, changing what a level costs, or changing what it does is a Data Asset job plus one small Blueprint.

Three folders hold everything:

- The upgrades: `Content/TheLastTemplate/Blueprints/DataAssets/Skills/Childs/`
- The effects they run: `Content/TheLastTemplate/Blueprints/Skills/Childs/`
- Who owns them: `Content/TheLastTemplate/Blueprints/ActorComponents/BP_PlayerSkillManager`, a component on `BP_PlayerCharacter`

---

## The three parts of an upgrade

**The Data Asset** is the upgrade itself, a child of `BP_SkillDataAsset`. It carries the name, the icon and the list of levels.

**The levels** are rows inside it. Each row is an `S_SkillLevel`: what it costs, what it says, and one number the effect reads.

**The effect class** is a child of `BP_SkillEffectBase`. It is the only Blueprint in the chain, and it does one thing: take the number from the level and write it somewhere on the player.

`Effect Classes` is an array, so one upgrade can drive several effects at once. All seven shipped upgrades use exactly one.

---

## The seven that ship

| Data Asset | What it changes | Levels, as `Cost` for `Value` |
|---|---|---|
| `DA_Skill_CraftingSpeed` | `Craft Speed Multiplier` on the inventory manager | 30 for 1.25, 60 for 1.5, 90 for 2 |
| `DA_Skill_HealingPower` | how much a health kit restores | 30 for 1.25, 60 for 1.5, 90 for 2 |
| `DA_Skill_StaminaRecovery` | how fast stamina comes back | 30 for 1.25, 60 for 1.5, 90 for 2 |
| `DA_Skill_SprintEndurance` | how fast stamina drains | 35 for 0.85, 70 for 0.7, 105 for 0.55 |
| `DA_Skill_DamageResistance` | incoming health damage | 40 for 0.9, 80 for 0.8, 120 for 0.7 |
| `DA_Skill_RecoilControl` | weapon recoil | 40 for 0.8, 80 for 0.6, 120 for 0.4 |
| `DA_Skill_CarryCapacity` | `Carry Bonus` on the inventory manager | 40 for 2, 80 for 4 |

Each one shows its `Display Name` on the page, in the same order: Crafting Speed, Healing Efficiency, Stamina Recovery, Sprint Endurance, Damage Resistance, Recoil Control, Carry Capacity.

`Value` is always a raw number the effect uses directly. Above 1 it multiplies up, below 1 it multiplies down, and Carry Capacity is the exception that adds a flat count instead. Nothing converts a percentage for you, so the level whose `Description` reads `Reduces weapon recoil by 60%.` holds `0.4`.

Carry Capacity is the only upgrade with two levels instead of three. Nothing forces three.

Five upgrades have their own icon in `Content/TheLastTemplate/Textures/Widgets/Icons/`, named `T_Skill_*`. `DA_Skill_CarryCapacity` and `DA_Skill_CraftingSpeed` reuse two icons drawn for other screens, `T_Parts` and `T_Crafting`. If you want them to match the others, draw two more.

---

## Change a level

1. Open the upgrade, for example `DA_Skill_RecoilControl`.
2. Open `Levels` and pick the row you want.
3. Edit the fields below.
4. Save.

| Field | What it does |
|---|---|
| `Title` | Level name. Empty on all seven shipped upgrades. |
| `Description` | The line the player reads on the row. Write the real effect here. |
| `Icon` | Per level icon. Unset on all seven shipped upgrades, which fall back to the upgrade's own `Icon`. |
| `Cost` | Supplements spent to buy this level. |
| `Craft Duration` | How long the player holds the button to learn it, in seconds. Shipped levels ramp 3, 3.5, then 4. |
| `Value` | The number handed to the effect class. |

Costs are absolute, not a price on top of the previous level. Level three of Recoil Control costs 120 supplements, not 120 more.

To add a level, add a row to `Levels` and fill the same six fields. To cut an upgrade down, delete rows. The page reads the array length, so a one level upgrade draws one pip and a five level upgrade draws five.

---

## Write a new effect

1. In `Content/TheLastTemplate/Blueprints/Skills/Childs/`, right click `BP_SkillEffectBase` and choose **Create Child Blueprint Class**. Name it `BP_SkillEffect_QuietSteps`.
2. Open it and override `ApplySkillEffect`.
3. The function hands you `Player Character`, `Level` and `Value`, and expects `Success` back.
4. Get the component you want from `Player Character`, write `Value` into it, then set `Success` to true.
5. Set `Success` to false if you could not apply it. The manager treats that as a failure.

`Level` is there for the rare effect that needs to branch instead of scale. Most of the shipped ones ignore it and use `Value` alone.

Then make the upgrade: right click in `Content/TheLastTemplate/Blueprints/DataAssets/Skills/Childs/`, then **Miscellaneous**, then **Data Asset**, pick `BP_SkillDataAsset` and name it `DA_Skill_QuietSteps`. Fill `Display Name`, `Icon`, add your levels, and put your new class in `Effect Classes`.

!!! note
    The class name does not have to match the Data Asset name. `DA_Skill_CraftingSpeed` points at `BP_SkillEffect_CraftSpeed`, and that is fine.

---

## Make it show up

A new upgrade does not appear on the Skills page on its own. The player has to know it. There are two ways.

**From the start.** Open `BP_PlayerCharacter`, select the `BP_PlayerSkillManager` component and add a row to `Known Skills`. Each row is an `S_SkillProgress`:

| Field | What it does |
|---|---|
| `Skill` | The upgrade Data Asset. |
| `Unlocked Levels` | How many levels the player is allowed to buy. |
| `Current Level` | How many are already bought. Leave at 0 for a fresh start. |

Four upgrades ship in that list: Crafting Speed, Damage Resistance and Sprint Endurance with `Unlocked Levels` at 99, and Carry Capacity at 1. Ninety nine simply means more levels than the upgrade has, so the whole line is open. Carry Capacity opens with one of its two levels and the second stays out of reach until something unlocks it.

**Found in the world.** Three upgrades are not in `Known Skills`: Healing Efficiency, Recoil Control and Stamina Recovery. The player learns them from a book they inspect. See [Choose what the two inspect buttons do](inspect_actions_and_notes.md) for how an inspect action is attached to an object.

The action class for it is `BP_InspectAction_UnlockSkill`. Make a child of it, set the two fields below, and point a book at that child:

| Field | What it does |
|---|---|
| `Skill to Unlock` | The upgrade Data Asset this book teaches. |
| `Unlocked Levels` | How many of its levels the book opens. |

The three shipped children sit in `Content/TheLastTemplate/Blueprints/Inspect/Actions/Childs/UnlockSkill/Childs/`. `BP_InspectAction_UnlockSkill_HealingPower` opens 3 levels at once. `BP_InspectAction_UnlockSkill_RecoilControl` and `BP_InspectAction_UnlockSkill_StaminaRecovery` open 1 each, so levels two and three of those two upgrades stay locked until something else opens them.

---

## Unlocked is not bought

Three numbers describe one upgrade for one player, and they are easy to confuse.

- `Levels` on the Data Asset is how many levels exist at all.
- `Unlocked Levels` is how far up the line the player is allowed to go.
- `Current Level` is how many are paid for.

An upgrade with `Unlocked Levels` at 0 is known but frozen. An upgrade absent from the list entirely is not on the page at all. That is the difference between an upgrade the player can see but not reach, and one they do not know exists.

---

## The currency

Supplements are an ordinary inventory item, not a separate score. The `Supplement Item` field on `BP_PlayerSkillManager` points at `DA_Item_Supplement`, whose `Category` is `Supplement` and whose `Max Carry` is 9999. The counter in the Skills page header just reads how many of that item are in the backpack.

To pay with something else, point `Supplement Item` at another item Data Asset. Any child of `BP_ItemDataAsset` works, including `DA_Item_Parts`. See [Add an item](../inventory/add_an_item.md) to make your own.

!!! warning
    Changing `Supplement Item` does not change what the pickups in your level hand out. Repoint them too, or the counter sits at zero forever and every level refuses to be bought with no message about why.

While you are placing supplements: `BP_Interactable_Pickup_Item_Supplement` ships with `SM_Explosive` as its mesh, the same one the explosive pickup uses. Give it your own mesh, as covered in [Place the other pickups](../inventory/place_other_pickups.md).

---

## The Skills page

The page is `BP_InventorySkillsPageWidget`, driven by `DA_Page_Skills` like every other backpack page. One row per known upgrade, `BP_SkillRowWidget`, and one pip per level, `BP_SkillPipWidget`. The header holds the supplement icon and count.

You do not need to touch any of it to add an upgrade. Add the Data Asset, put it in `Known Skills` or behind a book, and the row draws itself. If you want to change the page order or its icon, that is [Add an inventory page](../inventory/add_an_inventory_page.md).

---

## Refuse a purchase with your own rule

The manager already blocks a purchase when the player is short on supplements or is at the top of the line. To add a rule of your own, open `BP_PlayerSkillManager` and fill in `GetExtraLearnBlockReason`. `GetLearnBlockReason` calls it and passes its `Reason` straight through, so you never touch the built in checks or the widget.

That is where a rule like "only at a workbench" or "not while wounded" belongs.

---

## What gets saved

`SG_TLTGameSave` stores the whole `Skill Progress` array, so both `Unlocked Levels` and `Current Level` survive a reload for every upgrade the player knows. Nothing else about upgrades is saved, because nothing else changes at runtime. On load, the manager reapplies every bought level to the player. See [Saving a run](..

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
