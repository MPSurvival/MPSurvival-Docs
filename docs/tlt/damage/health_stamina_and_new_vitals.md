# Change health and stamina, or add a new vital

A vital is a number that goes down and comes back up. Health and stamina ship with the template, and the system is written so a third one, hunger or thirst or radiation, is a Data Asset and a row in an array.

Every vital is described by one Data Asset. Every character that can be hurt carries one `BP_VitalsSystem` component holding the list of vitals it owns.

- The component: `Content/TheLastTemplate/Blueprints/ActorComponents/BP_VitalsSystem`
- The Data Asset class: `Content/TheLastTemplate/Blueprints/DataAssets/Vitals/BP_VitalData`
- The two that ship: `Content/TheLastTemplate/Blueprints/DataAssets/Vitals/Childs/`
- The list of types: `Content/TheLastTemplate/Blueprints/Enumerations/Vitals/E_VitalsType`

---

## What ships

Two Data Assets, one of type Health and one of type Stamina.

| Data Asset | `Vital Max Amount` | `Vital Tick Decrementation` |
|---|---|---|
| `DA_Vital_Health` | `100` | `0` |
| `DA_Vital_Stamina` | `200` | `2` |

Four Blueprints carry the component:

| Blueprint | Starts with |
|---|---|
| `BP_PlayerCharacter` | health and stamina |
| `BP_NPCCharacter` | health |
| `BP_ZombieCharacter` | health |
| `BP_Dummy` | health |

Only the player has stamina. Nothing stops you from giving an NPC one, but nothing spends it either.

---

## The six fields of a vital

Open `DA_Vital_Health` and you see the whole of `BP_VitalData`.

| Field | What it does |
|---|---|
| `Vital Name` | A label. Free text. |
| `Vital Icon` | A texture for your own UI. |
| `Vital Type` | Which vital this is, picked from `E_VitalsType`. This is how the rest of the game finds it. |
| `Vital Color` | A colour for your own UI. |
| `Vital Max Amount` | The ceiling. `Add vital amount` never goes past it. |
| `Vital Tick Decrementation` | How much is taken every tenth of a second while the vital is ticking. |

`Vital Name`, `Vital Icon` and `Vital Color` are there for you. Nothing in the shipped HUD reads them, so leave them empty or fill them in, either is fine.

`Vital Type` is the important one. Health, damage, healing items, the death check and the HUD all look a vital up **by type**, and they take the first match in the list. Two vitals of the same type means the second one can never be reached.

---

## Change max health or max stamina

1. Open `Content/TheLastTemplate/Blueprints/DataAssets/Vitals/Childs/DA_Vital_Health`.
2. Set `Vital Max Amount` to the value you want.
3. Save.
4. Open `BP_PlayerCharacter`, select the `BP_VitalsSystem` component, and in `Vitals` set `Current Amount` on the health row to the same value.
5. Repeat step 4 on `BP_NPCCharacter`, `BP_ZombieCharacter` and `BP_Dummy` if you want them to follow.

!!! warning "`Current Amount` is not filled in from `Vital Max Amount`"
    The starting value of each vital is authored per actor, in the `Vitals` array on the component. Nothing copies `Vital Max Amount` into it at Begin Play. Raise health to 200 and forget step 4 and the character still spawns at 100, with a health bar sitting at half.

---

## The drain tick, and how to stop it

The component starts a repeating timer at Begin Play. Every tenth of a second it walks the `Vitals` array and, for each row whose `Is Paused Tick` is unchecked, removes `Vital Tick Decrementation` from it.

Health ships at `0`, so health never drains on its own. Stamina ships at `2`, which is 20 a second, so a full 200 empties in ten seconds of draining.

Each row of `Vitals` has three fields:

| Field | What it does |
|---|---|
| `Vital Asset` | The `BP_VitalData` this row uses. |
| `Current Amount` | What the character starts with. |
| `Is Paused Tick` | Checked means this vital is not draining right now. |

There are three ways to stop a drain:

- `Set vital pause tick`, with a `Type` and a `Pause` boolean, freezes one vital and leaves the others running. This is what the player character uses to hold stamina still when you are not sprinting.
- `Pause tick decrementation` and `Start tick decrementation` stop and restart the whole timer, every vital at once.
- Setting `Vital Tick Decrementation` to `0` on the Data Asset means the tick still runs but takes nothing.

---

## Give one character its own values

Two ways, and they are not the same.

**Different starting amount, same ceiling.** Select the actor, select its `BP_VitalsSystem` component, and change `Current Amount` on the row. A weakened NPC that starts at 40 of 100 needs nothing else.

**Different ceiling.** Duplicate `DA_Vital_Health`, name it for what it is, set its `Vital Max Amount`, then point that actor's `Vital Asset` at the new Data Asset. Keep `Vital Type` on Health so damage, death and healing still find it.

A placed actor works too. Select the instance in the level and edit `Vitals` on it, and only that one is affected.

---

## Add a third vital

Say you want hunger.

1. Open `Content/TheLastTemplate/Blueprints/Enumerations/Vitals/E_VitalsType` and add an enumerator. Set its display name to `Hunger`. Save.
2. Duplicate `DA_Vital_Stamina`, name it `DA_Vital_Hunger`. Set `Vital Type` to Hunger, `Vital Max Amount` to what you want, and `Vital Tick Decrementation` to how fast hunger should fall every tenth of a second.
3. Open `BP_PlayerCharacter`, select the `BP_VitalsSystem` component, add a row to `Vitals`. Set `Vital Asset` to `DA_Vital_Hunger`, `Current Amount` to the starting value, and leave `Is Paused Tick` unchecked so it drains.
4. Feed it. An item that restores hunger is a `BP_ItemDataAsset` whose effect calls `Add vital amount` with type Hunger, the way `BP_ItemEffect_Heal` does for health. See [Add a new item](../inventory/add_an_item.md).

That is a working vital. Be honest with yourself about what it does not do yet:

- it drains and refills, and it clamps at `0` and at `Vital Max Amount`, but reaching `0` does nothing. Only the Health type kills.
- none of the four multipliers apply to it. Health gets `Healing Multiplier` and `Damage Taken Multiplier`, stamina gets `Stamina Recovery Multiplier` and `Stamina Drain Multiplier`, a third vital gets neither.
- nothing draws it. The HUD is built for the two that ship, so you add the bar yourself.

If you want hunger to hurt you when it empties, remove health from your own logic when it hits `0`. There is no built in link between vitals.

---

## The four multipliers

They live on the component itself, all shipped at `1`, all category `Skills`.

| Field | What it scales | Skill that drives it |
|---|---|---|
| `Damage Taken Multiplier` | Every point of health removed. | `DA_Skill_DamageResistance` |
| `Healing Multiplier` | Every point of health added. | `DA_Skill_HealingPower` |
| `Stamina Recovery Multiplier` | Every point of stamina added. | `DA_Skill_StaminaRecovery` |
| `Stamina Drain Multiplier` | The stamina drain tick only. | `DA_Skill_SprintEndurance` |

Set them by hand for a difficulty setting, or leave them to the skill effects. Each skill in `Content/TheLastTemplate/Blueprints/DataAssets/Skills/Childs/` has a matching effect Blueprint that writes one of these when the player buys a level. See [Add or change a skill](../progression/add_or_change_a_skill.md).

`Invulnerable`, also on the component, blocks the death step. It does not block damage.

---

## Read a vital from your own widget

`Get vital` takes a `Type` and gives you back the row, which you break into `Vital Asset`, `Current Amount` and `Is Paused Tick`. Divide `Current Amount` by the `Vital Max Amount` of the `Vital Asset` and you have a percent for a progress bar. That is what `BP_LifeCharacterWidget` does for the health ring and the stamina bar. See [The HUD widgets](../ui/hud_widgets.md).

The functions you will use from outside:

| Function | What it does |
|---|---|
| `Get vital` | The row for a type. Read only. |
| `Add vital amount` | Adds, applies the healing or recovery multiplier, clamps at `Vital Max Amount`. |
| `Remove vital amount` | Removes, applies `Damage Taken Multiplier` on health, clamps at `0`, and runs the death check. |
| `Change vital amount` | Writes a value straight in. No multiplier, no clamp, no death check. |
| `Set vital pause tick` | Freezes or unfreezes one vital's drain. |
| `Get is dead` | Whether this character has already died. |
| `Set invulnerable state` | Turns the death check off and on. |

Two event dispatchers fire on the component: `On Health Damage` every time health drops, and `On Death` the moment it reaches `0` on a vulnerable character. Bind to them instead of polling.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
