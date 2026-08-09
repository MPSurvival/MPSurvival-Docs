# Add a collectible and a new category

A collectible is an object the player finds in the level, turns around in 3D, then keeps. When it is taken it disappears from the world and appears on the **Collectibles** page of the backpack, under a group header, with a dot next to it until the player opens it once. The counter at the top of the page goes up. What the player has found is saved.

The template ships three trading cards built exactly this way.

Adding your own is two Data Assets, one row on an inspect sheet, and one line added to an array. No graph.

---

## The pieces

| Asset | What it is | Where |
|---|---|---|
| `BP_InspectableDataAsset` | The 3D sheet: which mesh is shown, how it is framed, and the two buttons offered while looking at it. | `Content/TheLastTemplate/Blueprints/DataAssets/Inspect/Childs/` |
| `BP_CollectibleDataAsset` | The catalogue entry: a name, a category, and the inspect sheet it belongs to. | `Content/TheLastTemplate/Blueprints/DataAssets/Collectibles/Childs/` |
| `BP_CollectibleCategoryDataAsset` | A group header on the page. Several entries share one. | `Content/TheLastTemplate/Blueprints/DataAssets/Collectibles/Childs/` |

Plus one array, `All Collectibles`, on `Content/TheLastTemplate/Blueprints/ActorComponents/BP_PlayerInventoryManager`. That array is the list the player is asked to complete.

The entry and the sheet are separate on purpose. An inspectable object does not have to be a collectible, and the sheet is what the page reuses when the player wants to look at the card again from the backpack.

---

## Step 1, make the inspect sheet

If your object is not inspectable yet, do that first: see [Add an object the player can inspect](add_an_inspectable_object.md). In short, right click in `Content/TheLastTemplate/Blueprints/DataAssets/Inspect/Childs/`, then **Miscellaneous**, then **Data Asset**, and pick `BP_InspectableDataAsset`.

| Field | What it does |
|---|---|
| `Display Name` | The name written on screen while the object is held. The three cards all use `Trading Card`. |
| `Mesh` | The static mesh shown in the inspect view. |
| `Inspect Rotation` | The starting angle, as pitch, yaw and roll. The cards use `0 / 90 / 90`, which lands them face on. |
| `Frame Fill` | How much of the frame the object fills, `0` to `1`. The cards use `0.82`. |
| `Actions` | The buttons. This is where step 2 happens. |

---

## Step 2, add the Take action

`Actions` is an array of rows. Add one and set:

| Field | Value |
|---|---|
| `Action Class` | `BP_InspectAction_TakeCollectible` |
| `Slot` | `Secondary`, so it lands on the second inspect button |
| `Label` | `Take` |
| `Closes Inspect` | `true` |

Leave `Text Param`, `Text Lines`, `Float Param`, `Int Param` and `Class Param` alone. This action reads none of them.

Nothing on this row says which collectible is taken. The action looks at the sheet you are currently inspecting and finds the entry that points back at it. The link is the `Inspect Data` field in step 3, and it is the only link there is.

When the action runs it adds the entry to the player, then destroys the actor in the level.

The three cards also carry a `Read` action in the `Primary` slot, so the first button reads the card and the second one takes it. See [Choose what the two inspect buttons do](inspect_actions_and_notes.md) for the other actions and for writing your own.

---

## Step 3, make the collectible entry

Right click in `Content/TheLastTemplate/Blueprints/DataAssets/Collectibles/Childs/`, then **Miscellaneous**, then **Data Asset**, and pick `BP_CollectibleDataAsset`. It has three fields.

| Field | What it does |
|---|---|
| `Display Name` | The line printed in the catalogue. `The Comet`, `Hollow`, `Howler` on the shipped cards. |
| `Category` | The group it is filed under. Step 4. |
| `Inspect Data` | The sheet from step 1. This is what ties the entry to the object in the world, and what the page reopens when the player inspects the card again. |

---

## Step 4, choose or create the category

Categories are the group headers on the page. If your object belongs with an existing group, point `Category` at it and skip the rest of this step.

For a new group, make a Data Asset from `BP_CollectibleCategoryDataAsset`.

| Field | What it does |
|---|---|
| `Display Name` | The header text above the group. |
| `Icon` | A texture drawn next to the header. |

The shipped category is `DA_CollectibleCategory_TradingCards`, with `Display Name` set to `Trading Cards`. Its `Icon` is left empty, so the header shows text only. Set one if you want the icon slot filled.

Categories do not need to be registered anywhere. The page builds its groups from the categories the entries point at, so a category with no entry simply never appears.

---

## Step 5, register the entry

1. Open `Content/TheLastTemplate/Blueprints/ActorComponents/BP_PlayerInventoryManager`.
2. Open **Class Defaults**.
3. Add your entry to `All Collectibles`.
4. Compile and save.

!!! warning "An entry that is not in `All Collectibles` does nothing"
    It will not show on the page, it will not count, and the `Take` button will find nothing to give. Nothing errors, the button just does not work. This is the one step that is easy to forget.

---

## Step 6, put it in the level

There is no actor made just for collectibles. Drop a `BP_Interactable_Pickup_InspectableBase` into the level and set two fields on it:

| Field | Value |
|---|---|
| `Static Mesh` | The mesh the player sees lying there. It can be the same one the sheet uses. |
| `Inspect Data` | The sheet from step 1. |

`Use Icon` is on by default, so the prompt shows the `Interact Icon`, `T_Journal`, and not the `Interact Text`, `Examine`. Change either one per instance. If the actor shows an engine cube in the viewport, `Static Mesh` is still empty.

The three cards in `L_ShowcaseMap` are placed this way, as plain instances of the base class, not as child Blueprints.

---

## What the player sees

The **Collectibles** page is one of the backpack pages, described by `DA_Page_Collectibles`. It is already installed, so you never touch it to add an entry.

- Entries are listed under their category header.
- The top of the page shows how many entries are found out of the total in `All Collectibles`.
- A found entry that has never been opened carries a dot. Opening it clears the dot.
- The same header also shows how many Parts the player carries, read from `DA_Item_Parts`. Parts are what a weapon upgrade costs at the workbench. See [Add a weapon upgrade](../weapons/add_a_weapon_upgrade.md).
- Selecting an entry and pressing the inspect prompt reopens the 3D view, using the entry's `Inspect Data`. This is why the sheet is a separate asset.

Entries the player has not found are still listed, so the page reads as a checklist.

---

## What is saved

Two things go into the save: the entries the player has collected, and the ones still marked as unread. Both are stored as references to your Data Assets, so nothing else has to be told about a new entry.

---

## The three cards, end to end

Open these side by side when you build your first one.

| `Display Name` | Entry | Its `Inspect Data` |
|---|---|---|
| `The Comet` | `DA_Collectible_Card_Comet` | `DA_Inspect_Card_Comet` |
| `Hollow` | `DA_Collectible_Card_Hollow` | `DA_Inspect_Card_Hollow` |
| `Howler` | `DA_Collectible_Card_Howler` | `DA_Inspect_Card_Howler` |

All three entries have the same `Category`, `DA_CollectibleCategory_TradingCards`, and each sheet points at its own mesh: `SM_Card_Comet`, `SM_Card_Hollow` and `SM_Card_Howler`.

All three sheets are identical apart from the mesh and the text: `Display Name` `Trading Card`, `Inspect Rotation` `0 / 90 / 90`, `Frame Fill` `0.82`, a `Read` action on `Primary` and a `Take` action on `Secondary`.

The card faces are static meshes with their own material instances, so a fourth card means a mesh and a material, not a new system.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
