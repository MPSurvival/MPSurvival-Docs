# Add a new item to the inventory

By the end of this page you have an item that sits in the backpack with a name and an icon, a pickup the player can find in the level and take, and something that happens when the item is used.

Two folders hold the main pieces:

- the item: `Content/TheLastTemplate/Blueprints/DataAssets/Inventory/Childs/`
- the pickup that puts it in the world: `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/Items/Childs/`

---

## One item is one Data Asset

Twelve items ship with the template and each one is a single Data Asset made from `BP_ItemDataAsset`. There is no item actor and no graph to edit.

1. In `Content/TheLastTemplate/Blueprints/DataAssets/Inventory/Childs/`, right click, then **Miscellaneous**, then **Data Asset**.
2. Pick `BP_ItemDataAsset` as the class.
3. Name it `DA_Item_Bandage`, or whatever your item is.
4. Fill the fields below.
5. Save.

## The fields on the item Data Asset

| Field | What it does |
|---|---|
| `Display Name` | The name shown in the bag. |
| `Description` | One line under it, for example "Cloth scraps." on the rag and "Flammable liquid." on the alcohol. |
| `Icon` | The texture drawn in the slot. |
| `Icon Scale` | Shrinks or grows that texture inside the slot. `1` on every shipped item. |
| `Max Carry` | How many of this item the player can hold. |
| `Stackable` | Whether several of them share one slot. `true` on every shipped item. |
| `Effect Amount` | A single number the effects can use. `50` on the health kit, `0` on every other shipped item. |
| `Use Montage` | Animation to play when the item is used. Empty on every shipped item. |
| `Use Sound` | Sound to play when the item is used. Empty on every shipped item. |
| `Pickup Class` | The pickup actor that stands for this item in the world. |
| `Effect Classes` | The list of things that happen when the item is used. |
| `Category` | One of four values, see below. |
| `Auto Use` | Runs the effects as soon as the item is picked up. |
| `Throwable Data` | The throwable this item stands for, if it stands for one. |

`Pickup Class` points at the matching pickup on every shipped item except the molotov and the nail bomb, which leave it empty and carry a `Throwable Data` instead.

## The four categories

| `Category` | Shipped items |
|---|---|
| `Consumable` | Health Kit, Molotov, Nail Bomb |
| `CraftingMaterial` | Alcohol, Binding, Blade, Explosive, Parts, Rag, Sugar |
| `KeyItem` | Journal |
| `Supplement` | Supplements |

Two of the shipped items are currencies rather than things you use. `Parts` is what the workbench spends on weapon upgrades, and it is filed as a `CraftingMaterial` with a `Max Carry` of `999`. `Supplements` is what the skill page spends, it has its own category, and its `Max Carry` is `9999`. If you add a currency of your own, copy that shape: a high `Max Carry` and no `Effect Classes`.

---

## Give the item an effect when it is used

`Effect Classes` is a list, so one item can run several effects one after the other. Each entry is a child of `BP_ItemEffectBase`, which sits in `Content/TheLastTemplate/Blueprints/Inventory/`.

Two effects ship:

- `BP_ItemEffect_Heal` adds to a vital through the player's `BP_VitalsSystem`. This is the one the health kit uses.
- `BP_ItemEffect_GiveThrowable` hands a throwable to the player's `BP_PlayerThrowableManager`. It has one field, `Throwable Data`. Its two children, `BP_ItemEffect_GiveThrowable_Molotov` and `BP_ItemEffect_GiveThrowable_NailBomb`, set that field and add nothing else.

To write your own:

1. In `Content/TheLastTemplate/Blueprints/Inventory/Childs/`, create a Blueprint Class with `BP_ItemEffectBase` as the parent. Name it `BP_ItemEffect_Bandage`.
2. Open it and override `Execute Effect`. It gives you the `Player Character` and an `Amount`, and wants a `Success` back.
3. Do your work there, then set `Success`.
4. Add the class to `Effect Classes` on your item Data Asset.

If your effect only differs from an existing one by a value, do what the two throwables do: make a child of the effect you already have, set the one field, leave the graph alone.

---

## Make the item findable in the level

The item Data Asset does not place anything. A pickup does.

1. In `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/Items/Childs/`, create a Blueprint Class with `BP_Interactable_Pickup_Item` as the parent. Name it `BP_Interactable_Pickup_Item_Bandage`. Duplicating one of the ten that ship is faster.
2. Set `Item Data` to your Data Asset.
3. Set `Amount` to how many one pickup gives.
4. Set `Static Mesh` to the mesh you want lying on the floor.
5. Set `Interact Text`, or turn `Use Icon` on and set `Interact Icon`.
6. Go back to the item Data Asset and set `Pickup Class` to this new class.
7. Drag it into your level.

| Field | What it does |
|---|---|
| `Item Data` | The item this pickup gives. |
| `Amount` | How many. `1` on most shipped pickups, `8` on Parts, `10` on Supplements. |
| `Static Mesh` | The mesh shown in the world. |
| `Use Physics` | Lets the pickup fall and settle instead of floating where you placed it. On by default. |
| `Use Overlay Material` | The highlight material that makes the pickup readable in a dark room. On by default. |
| `Show Amount` | Adds the count to the prompt. On for item pickups. |
| `Interact Text` | The word on the prompt. |
| `Use Icon` and `Interact Icon` | Show an icon instead of the word. |
| `Error Text` | The text shown when the pickup refuses. `Full` by default. |
| `Interact Zone Range` | How close the player must be to take it. `100` on pickups. |
| `Show Interact Zone Range` | How close the player must be to see the prompt. `200` on pickups. |

The last five come from the interaction base and behave the same on every interactable, so they are explained once in [How interaction works](how_interaction_works.md).

---

## Start the player with the item

`BP_PlayerInventoryManager` is a component on `BP_PlayerCharacter`, and its `Slots` array is the bag the player starts the game with. Each row is one `Item` and one `Count`. Add a row, pick your Data Asset, type a number.

`Known Recipes` on the same component is the same idea for crafting: the recipes the player already knows on the first frame.

---

## Carry limit and a full bag

`Max Carry` on the item is the base limit. The shipped numbers are `10` for the crafting materials, `3` for the health kit and the two throwables, `1` for the journal, and the two large ones on `Parts` and `Supplements`.

On top of that, `BP_PlayerInventoryManager` has a `Carry Bonus`. The Carry Capacity skill raises it, so a player who trains carries more of everything. `Get Carry Limit` is the function that answers with the final number, and `Add Item` returns `Remaining`, which is what did not fit.

A pickup the player cannot take shows a reason rather than the normal prompt. That is the `Full` you see on a pickup when the bag is at the limit. Skills are covered in [Add or change a skill](../progression/add_or_change_a_skill.md).

---

## Items that use themselves at once

`Auto Use` is on for the molotov and the nail bomb only. Both of them hand a throwable to the throwable manager through their `Effect Classes`, so what the player picks up goes straight to the throw slot instead of waiting in the bag. Use it for anything that should take effect the moment the player touches it.

The throwables themselves, and the pickups that place them directly, are in [Place ammo, weapon, throwable and gear pickups](place_other_pickups.md) and [Add your own throwable](../throwables/add_your_own_throwable.md).

---

## Add your own reason to block a use

`BP_PlayerInventoryManager` keeps its refusals in pairs. One function holds the rules that ship, and the one beside it is left for yours:

| Rules that ship | Where yours go |
|---|---|
| `Get Use Block Reason` | `Get Extra Use Block Reason` |
| `Get Craft Block Reason` | `Get Extra Craft Block Reason` |
| `Get Collect Block Reason` | `Get Extra Collect Block Reason` |

Each returns a `Reason`. Put your test in the `Extra` one and the shipped rules stay as they are, which keeps your work separate the next time you open the component.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
