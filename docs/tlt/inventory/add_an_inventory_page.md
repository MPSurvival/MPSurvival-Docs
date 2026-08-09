# Add your own page to the inventory screen

The inventory screen ships with three pages: crafting, skills and collectibles. None of them is wired in by name. Each one is a widget, plus a Data Asset, plus one row in a list. A fourth page of your own is added exactly the same way, and you never open the inventory screen graph to do it.

- The screen: `Content/TheLastTemplate/Blueprints/Widgets/Inventory/BP_InventoryWidget`
- The page base you inherit from: `Content/TheLastTemplate/Blueprints/Widgets/Inventory/BP_InventoryPageBase`
- The page Data Assets: `Content/TheLastTemplate/Blueprints/DataAssets/Inventory/Childs/`

---

## What the screen is made of

`BP_InventoryWidget` holds three things that matter to you, and nothing else.

| Part | What it is |
|---|---|
| The tab strip | One `BP_InventoryTabWidget` per page, built at run time from the page list |
| The page switcher | One live widget per page, showing the current one |
| The prompt bar | A single `BP_MenuPromptBarWidget` at the bottom, shared by every page |

The screen changes page, plays the navigation sounds and passes input down. It does not know what a recipe, a skill or a collectible is. All of that lives inside the page widgets.

---

## The page list

`BP_InventoryWidget` has one variable you will edit: `Pages`. It is an array of `BP_InventoryPageDataAsset` and it ships with three entries, in this order.

| Data Asset | `Display Name` | `Page Icon` |
|---|---|---|
| `DA_Page_Craft` | Crafting | `T_Crafting` |
| `DA_Page_Skills` | Skills | `T_Supplement` |
| `DA_Page_Collectibles` | Collectibles | `T_Journal` |

Order in the array is the order of the tabs, left to right. Remove a row and that page and its tab are gone, with no other change anywhere.

The Data Asset itself has three fields.

| Field | What it does |
|---|---|
| `Display Name` | The name of the page |
| `Page Icon` | The texture drawn on the tab |
| `Page Widget Class` | The widget class the switcher creates for this page |

---

## The page base

`BP_InventoryPageBase` is a `User Widget` with five functions. Three of them return nothing, so when you override them in a child they show up in the Event Graph as events. Two of them return a boolean, so they show up as functions.

| Function | When the screen calls it | What you do in it |
|---|---|---|
| `Setup Page` | Once, just after your widget is created | The base stores the screen that created it in `Owner Screen`. Add your own one time setup after that. |
| `Refresh Page` | Whenever the screen wants the page redrawn, including when the player switches to it | Rebuild your rows, your text and your icons from the current game state |
| `Build Page Prompts` | When your page becomes the current one | Call `Clear Prompts` on the prompt bar you are handed, then `Add Prompt` once per action your page offers |
| `Handle Page Prompt` | When the player clicks a prompt in the bar | Compare the `Label` you receive with the label you passed to `Add Prompt`. Do the action and return true. |
| `Handle Page Key` | When a key is pressed while the screen is open | Act on the key you care about and return true. Return false and the screen keeps the key for its own tab navigation. |

Override only what your page needs. An empty override is not required, and the shipped pages do not all use the same set.

---

## Build the page widget

1. In `Content/TheLastTemplate/Blueprints/Widgets/Inventory/`, right click, then **User Interface**, then **Widget Blueprint**.
2. When Unreal asks for the parent class, pick `BP_InventoryPageBase`, not `User Widget`. Name it `BP_InventoryMapPageWidget`.
3. Lay the page out in the Designer. You get the whole area under the tab strip.
4. In the **My Blueprint** panel, use the **Override** dropdown next to **Functions** and pick `Refresh Page`. Fill it with whatever reads your data and pushes it into your widgets.
5. If your page needs a button at the bottom, override `Build Page Prompts` and `Handle Page Prompt` as well.
6. Compile and save.

To read what the player owns, get the player character and its `BP_PlayerInventoryManager` component. That component holds the item slots, the known recipes and the collected collectibles. `Owner Screen` gets you back to the inventory screen if you need it.

---

## Make the page Data Asset

1. In `Content/TheLastTemplate/Blueprints/DataAssets/Inventory/Childs/`, right click, then **Miscellaneous**, then **Data Asset**.
2. Pick `BP_InventoryPageDataAsset` as the class. Name it `DA_Page_Map`.
3. Set `Display Name`.
4. Set `Page Icon`. The tab icons that ship live in `Content/TheLastTemplate/Textures/Widgets/Icons/`, and yours can go there too.
5. Set `Page Widget Class` to `BP_InventoryMapPageWidget`.
6. Save.

!!! warning "The widget class has to inherit from the page base"
    The screen casts every page widget it creates to `BP_InventoryPageBase`. If you point `Page Widget Class` at a plain `User Widget`, the tab appears, the page stays blank, and nothing is printed to the log.

---

## Add the row to the list

1. Open `BP_InventoryWidget`.
2. Go to **Class Defaults**.
3. Find `Pages`, add an element, and set it to `DA_Page_Map`.
4. Drag the element up or down to decide where your tab sits.
5. Compile and save.

That is the whole wiring. Play, open the backpack, and your tab is there with the others.

---

## Match how the shipped pages feel

Three habits run through the existing pages. Copy them and your page will not feel bolted on.

- **Hovering selects.** Moving the mouse onto a row or a tile makes it the selection. There is no separate click to select.
- **Actions that take time are held, not clicked.** On the craft page you press and hold on a tile, and it fills over the recipe's `Craft Duration`. Releasing early cancels it.
- **Everything else is a prompt.** The label you pass to `Add Prompt` comes back to you in `Handle Page Prompt`. The label is the identity of the prompt, so keep it stable.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
