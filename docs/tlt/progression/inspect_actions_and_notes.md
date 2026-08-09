# Choose what the two inspect buttons do

While the player holds an object in the inspect view, the screen offers up to two buttons. What each button says, and what it does when pressed, comes from one array: `Actions`, on the inspect Data Asset of that object.

Five action classes ship. You can add your own without touching anything that already works.

- The sheets: `Content/TheLastTemplate/Blueprints/DataAssets/Inspect/Childs/`
- The action classes: `Content/TheLastTemplate/Blueprints/Inspect/Actions/`

If you have not read [How 3D inspection works](how_inspection_works.md) yet, start there.

---

## One row is one button

Open any `DA_Inspect_*` and look at `Actions`. Each row is one button.

| Field | What it does |
|---|---|
| `Action Class` | The Blueprint that runs when the button is pressed |
| `Slot` | `Primary` or `Secondary`. One row per slot |
| `Label` | The word written on the button |
| `Closes Inspect` | `true` leaves the inspect view after the action runs, `false` stays on the object |
| `Text Param` | Spare text |
| `Text Lines` | Spare list of texts |
| `Float Param` | Spare number |
| `Int Param` | Spare whole number |
| `Class Param` | Spare Actor class |

The last five are a bag of spare values. The row does not decide what they mean, the `Action Class` does. An action that has no use for a value simply never reads it, so leaving them empty is normal.

`Primary` is the first inspect key and `Secondary` the second. They ship on `IA_InspectAction1`, which is `E`, and `IA_InspectAction2`, which is `F`. You change them like any other key. See [Change the controls](../start/change_the_controls.md).

A mouse click on a prompt finds its row by `Label`, so give your two rows two different words.

---

## What each shipped action reads

| Action class | From the row | From its own class defaults |
|---|---|---|
| `BP_InspectAction_Read` | `Text Param`, `Text Lines` | |
| `BP_InspectAction_TakeItem` | `Class Param` | |
| `BP_InspectAction_TakeCollectible` | | |
| `BP_InspectAction_LearnRecipe` | | `Recipe to Learn` |
| `BP_InspectAction_UnlockSkill` | | `Skill to Unlock`, `Unlocked Levels` |

Two of them are set up on the row itself. Two are set up on a child class, one child per recipe and one child per upgrade, so several objects can share the same behaviour with different targets. `BP_InspectAction_TakeCollectible` carries nothing at all: it finds the catalogue entry from the sheet the player is looking at.

`Float Param` and `Int Param` are read by no shipped action. They are there for the actions you write.

---

## Read: put text on the screen

`BP_InspectAction_Read` opens a scrolling panel over the object. `Text Param` is the title, `Text Lines` is the body, one array entry per paragraph.

To write a note:

1. On your `DA_Inspect_*`, add a row to `Actions`.
2. Set `Action Class` to `BP_InspectAction_Read`.
3. Set `Slot`, and `Label` to the word you want on the button. The shipped notes use `Read`.
4. Put the title in `Text Param`.
5. Add one entry to `Text Lines` for each paragraph.
6. Leave `Closes Inspect` off. The panel is drawn over the inspect view, and closing it puts the player back on the object.

`DA_Inspect_Note_Sorry` is the smallest example: a single `Read` row, title `Note`, six paragraphs, nothing else.

!!! warning "The paper and the panel are two separate texts"
    The words printed on the mesh come from a texture, not from `Text Lines`. `SM_Note_Sorry` reads `T_Note_Sorry_BC` through `MI_Prop_Note_Sorry`. Edit `Text Lines` and the sheet of paper in the player's hands still says the old thing. Change both, or the note contradicts itself.

---

## Learn Recipe: teach a crafting recipe from the world

`BP_InspectAction_LearnRecipe` ships with `Recipe to Learn` empty on purpose. It is meant to be subclassed, one child per recipe.

1. In `Content/TheLastTemplate/Blueprints/Inspect/Actions/Childs/LearnRecipe/Childs/`, create a Blueprint Class with `BP_InspectAction_LearnRecipe` as parent.
2. In **Class Defaults**, set `Recipe to Learn` to your `DA_Recipe_*`.
3. Add a row on your sheet pointing at that child, with `Closes Inspect` on.

`DA_Inspect_Note_Recipe_Molotov` shows the pattern in full: `Learn` on `Primary` using `BP_InspectAction_LearnRecipe_Molotov` and closing the view, `Read` on `Secondary` staying open. Recipes themselves are covered in [Add a crafting recipe](../inventory/add_a_crafting_recipe.md).

---

## Unlock Skill: open upgrade levels from a book

Same shape, one child per book. `Skill to Unlock` picks the upgrade line, `Unlocked Levels` says how many of its levels become available to buy.

| Child class | `Skill to Unlock` and `Unlocked Levels` | Used by |
|---|---|---|
| `BP_InspectAction_UnlockSkill_HealingPower` | `DA_Skill_HealingPower`, `3` levels | `DA_Inspect_Book_Medical` |
| `BP_InspectAction_UnlockSkill_RecoilControl` | `DA_Skill_RecoilControl`, `1` level | `DA_Inspect_Book_Gunsmith` |
| `BP_InspectAction_UnlockSkill_StaminaRecovery` | `DA_Skill_StaminaRecovery`, `1` level | `DA_Inspect_Book_Field` |

Unlocking is not buying. The player still pays for each level on the upgrade page. See [Add or change a player upgrade](add_or_change_a_skill.md).

---

## Take Collectible: claim a catalogue entry

`BP_InspectAction_TakeCollectible` needs no setup at all. Drop it on a row, give it a `Label`, turn `Closes Inspect` on, and the object can be collected. It works out which catalogue entry belongs to the sheet the player is holding, and it blocks itself when that entry cannot be taken, so a card the player already owns does not offer the button again.

The three shipped trading cards use it on `Secondary`, with `Read` on `Primary`. The catalogue side of the job is on [Add a collectible and a new category](add_a_collectible.md).

---

## Take Item: hand over an ordinary pickup

`BP_InspectAction_TakeItem` reads `Class Param`. Set it to an interactable Blueprint, and the action spawns that Blueprint and runs its interaction straight away, so the player receives whatever it would have given on the ground.

No shipped sheet uses it. It is the row to add when the object being inspected is a plain item rather than a collectible, and you still want a `Take` button on it.

---

## Write your own action

1. Right click in `Content/TheLastTemplate/Blueprints/Inspect/Actions/Childs/`, create a Blueprint Class, and pick `BP_InspectActionBase` as parent.
2. Override `Execute Action`. You get `Inspector`, the player's `BP_PlayerInspectManager`, and `Params`, the row that fired. Break `Params` to read `Text Param`, `Int Param`, `Class Param` and the rest.
3. Do your work. Reach the rest of the game through the player who owns `Inspector`.
4. Add a row on a `DA_Inspect_*` with your class, a `Slot` and a `Label`.

!!! note "An action is spawned, used, and destroyed"
    Actions are Actors. The template spawns one when it needs to run or question it, and destroys it right after. Nothing you store on the action itself survives to the next press. Keep state on the player.

---

## Block an action and tell the player why

Two levers, both on the action class.

- Override `Get Block Reason`. It gets `Inspector` and returns `Reason`. Empty text means allowed, any text means blocked and that text is the explanation.
- `Hide When Blocked`, in **Class Defaults**. Every shipped action has it on, which removes the button entirely. Turn it off and the button stays on screen instead of disappearing.

For a rule that applies to every action at once, there is `Get Extra Action Block Reason` on `BP_PlayerInspectManager`. It ships empty and returns nothing. Fill it in and every prompt on every sheet obeys it, without editing a single action class.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
