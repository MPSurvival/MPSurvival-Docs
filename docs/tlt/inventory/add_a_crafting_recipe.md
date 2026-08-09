# Add a crafting recipe and teach it in the level

By the end of this page you have a recipe that shows up on the **Crafting** page of the backpack, spends items the player is carrying, and hands back a new item after a hold. You also have the choice of giving that recipe to the player from the first frame, or making them find a note in the level to learn it.

Everything lives in one folder: `Content/TheLastTemplate/Blueprints/DataAssets/Inventory/Childs/`.

---

## One recipe is one Data Asset

Three recipes ship: `DA_Recipe_HealthKit`, `DA_Recipe_Molotov` and `DA_Recipe_NailBomb`. Each one is a single Data Asset made from `BP_CraftRecipeDataAsset`. There is no recipe actor and no graph.

1. In `Content/TheLastTemplate/Blueprints/DataAssets/Inventory/Childs/`, right click, then **Miscellaneous**, then **Data Asset**.
2. Pick `BP_CraftRecipeDataAsset` as the class.
3. Name it `DA_Recipe_Bandage`, or whatever you are making.
4. Fill the fields below.
5. Save.

The result and the ingredients are ordinary item Data Assets. If the item you want to produce does not exist yet, make it first: [Add a new item to the inventory](add_an_item.md).

## The fields on the recipe Data Asset

| Field | What it does |
|---|---|
| `Display Name` | The name written on the recipe tile. |
| `Result` | The `DA_Item_*` the player gets. |
| `Result Count` | How many of it. `1` on all three shipped recipes. |
| `Ingredients` | The list of items spent, and how many of each. |
| `Craft Duration` | Seconds the player holds the click to make one. |
| `Requires Workbench` | See the warning below. |
| `Icon` | The texture drawn on the recipe tile. |
| `Craft Sound` | Played when the craft finishes. Empty on all three shipped recipes. |

What the three shipped recipes contain. The `Result` of each one is the item of the same name: `DA_Recipe_HealthKit` makes `DA_Item_HealthKit`, and so on.

| Recipe | Ingredients | `Craft Duration` |
|---|---|---|
| `DA_Recipe_HealthKit` | Rag 1, Alcohol 1 | `3` |
| `DA_Recipe_Molotov` | Rag 1, Alcohol 1 | `2.5` |
| `DA_Recipe_NailBomb` | Explosive 1, Binding 1 | `3` |

The health kit and the molotov are built from the same two materials. Change one of the two recipes if you want the player to have to choose between two different piles of loot.

!!! warning
    `Requires Workbench` is off on all three recipes and nothing in the template reads it. Crafting happens in the backpack, wherever the player is standing. Turning it on changes nothing on its own. If you want a recipe that only works near a bench, write that test in `Get Extra Craft Block Reason`, further down this page.

---

## Ingredients

`Ingredients` is an array of `S_CraftIngredient`. Each row has two fields:

| Field | What it does |
|---|---|
| `Item` | The `DA_Item_*` that gets spent. |
| `Count` | How many are taken per craft. `1` on every shipped row. |

Nothing stops you from using a `Consumable` as an ingredient, but the shipped recipes only spend items filed under the `CraftingMaterial` category. Seven items carry that category.

!!! warning
    The line under the recipe tile is built from two text slots, `Recipe Ing 1` and `Recipe Ing 2`, on `BP_InventoryCraftPageWidget`. Add a third ingredient and it is spent normally but never written on screen. Keep a recipe to two ingredients unless you also edit that widget.

---

## The materials bar at the top of the craft page

The row of icons across the top of the Crafting page is a hand written list, not an automatic sweep of every crafting material. It lives on `BP_InventoryCraftPageWidget`, in `Content/TheLastTemplate/Blueprints/Widgets/Inventory/`.

| Field | What it does | Shipped value |
|---|---|---|
| `Material Items` | The items shown in the bar, in order | Alcohol, Rag, Sugar, Explosive, Binding, Blade |
| `Materials Per Row` | How many fit on one line before it wraps | `6` |

Each entry shows the item icon and how many the player is carrying.

If you invent a new crafting material, add it to `Material Items` yourself. Leaving it out does not break anything, recipes using it still work, but the player has no way to see how many they have. Note that `Parts` is filed as a `CraftingMaterial` and is deliberately absent from this bar: it is the workbench currency, not a crafting ingredient. See [Put a workbench in your level](../weapons/put_a_workbench_in_your_level.md).

---

## Recipes known from the start, and recipes that must be learned

The craft page only lists recipes the player knows. That list is `Known Recipes` on `BP_PlayerInventoryManager`, the component on `BP_PlayerCharacter`.

It ships with two entries: `DA_Recipe_HealthKit` and `DA_Recipe_NailBomb`. `DA_Recipe_Molotov` is missing on purpose, because it is the one you learn from a note in the level.

- To give the player a recipe from the first frame, add it to `Known Recipes` and stop there.
- To make them find it, leave the array alone and put the recipe on a note.

Four functions on the component handle the rest: `Knows Recipe` answers yes or no, `Learn Recipe` adds one, `Forget Recipe` takes one away, and `Get Craftable Recipes` returns the ones the player can afford right now. `Learn Recipe` fires the `On Recipe Learned` dispatcher, which is where you hang a notification of your own.

---

## Put a recipe on a note in the level

A note is an inspectable object carrying a **Learn** button. It takes an action class, an inspect Data Asset and an actor placed in the level, in this order.

1. In `Content/TheLastTemplate/Blueprints/Inspect/Actions/Childs/LearnRecipe/Childs/`, create a Blueprint Class with `BP_InspectAction_LearnRecipe` as the parent. Name it `BP_InspectAction_LearnRecipe_Bandage`.
2. Open it and set `Recipe To Learn` to your recipe Data Asset. That is the only field to touch. `BP_InspectAction_LearnRecipe_Molotov` is exactly this and nothing more.
3. In `Content/TheLastTemplate/Blueprints/DataAssets/Inspect/Childs/`, create a Data Asset from `BP_InspectableDataAsset`. Set `Mesh` to the paper you want held up, `SM_Note_Recipe` for the shipped one.
4. Add a row to `Actions`: `Action Class` is the class from step 1, `Slot` is `Primary`, `Label` is the word on the button, and turn `Closes Inspect` on so the note is put away once the recipe is learned.
5. Optionally add a second row with `BP_InspectAction_Read` on `Slot` `Secondary`, so the player can also read the page. It takes `Text Param` as the title and `Text Lines` as the body, one string per paragraph.
6. Drag a `BP_Interactable_Pickup_InspectableBase` into the level, set its `Inspect Data` to the Data Asset from step 3, and set its `Static Mesh` to the same note mesh so it reads on the floor.

`DA_Inspect_Note_Recipe_Molotov` is the shipped example and follows exactly that shape: a **Learn** button and a **Read** button on the same sheet of paper.

`Hide When Blocked` on the action ships on, so a **Learn** button that cannot run is not drawn at all rather than sitting there dead. The inspectable side of this, the framing, the rotation and the other action classes, is covered in [Add an inspectable object](../progression/add_an_inspectable_object.md). The prompt fields on the actor itself behave like every other interactable, see [How interaction works](how_interaction_works.md).

---

## Change how long a craft takes

`Craft Duration` on the recipe is the base time in seconds. The player holds the click on the tile, a bar fills, and letting go early cancels with nothing spent.

`Craft Speed Multiplier` on `BP_PlayerInventoryManager` scales that at runtime. It ships at `1` and the Crafting Speed skill, `DA_Skill_CraftingSpeed`, raises it through `BP_SkillEffect_CraftSpeed`. So a recipe set to `3` is not always three seconds. Tune the recipe first and leave the multiplier to the skill: [Add or change a skill](../progression/add_or_change_a_skill.md).

---

## Add your own reason to block crafting

`BP_PlayerInventoryManager` keeps two functions side by side. `Get Craft Block Reason` holds the rules that ship, and `Get Extra Craft Block Reason` is the one left for you. Both return the reason, and the craft page shows it instead of letting the hold start.

Put your test in the `Extra` one. That is where a workbench requirement, a story flag or a time of day goes, and it keeps your work separate from the shipped rules the next time you open the component.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
