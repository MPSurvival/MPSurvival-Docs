# The developer menu

The template ships an in game debug menu. It hands you items, weapons and skills, spawns AI in front of you, and turns a few display helpers on and off, so you can test a change without playing up to it.

Open it with the `IA_DevMenu` action, mapped to `!` by default. Change that key like any other, in `IMC_Default`. See [Change the controls](../start/change_the_controls.md).

The widget is `Content/TheLastTemplate/Blueprints/Widgets/DevDebug/BP_DevDebugHUD`.

---

## Moving around

Categories are listed on the left, the rows of the open category on the right, with the description of the selected row underneath and a status line that confirms what just happened.

| Key | What it does |
|---|---|
| Arrow keys | Move between categories and rows |
| `Enter` | Run the selected row |
| `Esc` | Go back one level, then close the menu |

The mouse cursor is shown while the menu is open, and rows answer to hover and click.

The menu does not pause the game. Your movement and your camera are blocked while it is open, but the world keeps running, so AI you already spawned will keep coming at you while you read.

---

## Player

| Row | What it does |
|---|---|
| `HEAL TO FULL` | Restores health to maximum. |
| `RESTORE STAMINA` | Restores stamina to maximum. |
| `DEAL 25 DAMAGE` | Removes 25 health from the player. |
| `KILL PLAYER` | Drops invulnerability and kills the player. |
| `GOD MODE` | The player takes no damage at all. |
| `INFINITE STAMINA` | Cancels all stamina drain. |

`GOD MODE` and `INFINITE STAMINA` are toggles. They stay on until you run the row again. `KILL PLAYER` turns god mode off first, so it always kills you.

---

## Inventory

| Row | What it does |
|---|---|
| `CRAFTING MATERIALS` | Adds 25 of every item listed in `Material Items`. |
| `CONSUMABLES` | Adds 5 of every item listed in `Consumable Items`. |
| `THROWABLES` | Adds 5 of every item listed in `Throwable Items`. |
| `SUPPLEMENTS` | Adds 100 supplements. |
| `UNLOCK ALL` | Marks every collectible in the game as found. |
| `CLEAR ALL` | Empties the collection. |
| `EMPTY INVENTORY` | Removes every item from the bag. |

The first four rows respect the carry limit, so a bag that is already full will not take everything.

---

## Weapons

| Row | What it does |
|---|---|
| One row per weapon | Gives this weapon to the player and refreshes what is carried. |
| `INFINITE AMMO` | The magazine never runs out. |

The give rows are not written by hand. The category builds one row for every entry in `Weapon Classes`, named after the class. Add a weapon there and the row appears.

---

## Skills

| Row | What it does |
|---|---|
| `UNLOCK ALL SKILLS` | Makes every tier of every skill available. |
| `BUY ALL SKILLS` | Grants supplements, then learns every tier. |
| `RESET SKILLS` | Rolls every skill back to zero. |

`UNLOCK ALL SKILLS` only opens the tiers, it does not pay for them. `BUY ALL SKILLS` gives you the supplements first and then learns everything, which is what you want when you are testing a skill effect.

---

## AI

| Row | What it does |
|---|---|
| `STANDING NPC` | Spawns a stationary NPC in front of the player. |
| `ROAMING NPC` | Spawns a patrolling NPC in front of the player. |
| `MELEE NPC` | Spawns a melee NPC in front of the player. |
| `INFECTED` | Spawns an infected in front of the player. |
| `KILL ALL AI` | Applies lethal damage to every AI in the level. |

Every spawn drops the character a short distance ahead of you, facing the direction you were looking. `KILL ALL AI` does not delete anything, it applies damage, so the AI die through the normal death path.

---

## Display

| Row | What it does |
|---|---|
| `HIDE HUD` | Hides the gameplay widgets. |
| `PLAYER DEBUG` | Enables the character debug draws. |
| `FPS COUNTER` | Shows the frames-per-second counter. |
| `SHOW COLLISION` | Draws the collision volumes. |
| `PHOTO MODE` | Closes the menu and opens photo mode. |

All of these except `PHOTO MODE` are toggles. For what happens after the last one, see [Photo mode, tab by tab](photo_mode_overview.md).

---

## Change what the give rows hand out

The lists the menu draws from are variables on `BP_DevDebugHUD`. Open the Blueprint, select the variable in **My Blueprint**, and edit its default value in the **Details** panel. No graph work.

| Field | What it feeds | Ships with |
|---|---|---|
| `Material Items` | `CRAFTING MATERIALS`, 25 of each | `DA_Item_Alcohol`, `DA_Item_Binding`, `DA_Item_Blade`, `DA_Item_Explosive`, `DA_Item_Rag`, `DA_Item_Sugar` |
| `Consumable Items` | `CONSUMABLES`, 5 of each | `DA_Item_HealthKit` |
| `Throwable Items` | `THROWABLES`, 5 of each | `DA_Item_Molotov`, `DA_Item_NailBomb` |
| `Supplement Item` | `SUPPLEMENTS` | `DA_Item_Supplement` |
| `Weapon Classes` | One give row each, in `WEAPONS` | `BP_Weapon_Pistol_01`, `BP_Weapon_Pistol_02`, `BP_Weapon_Shotgun_01`, `BP_Weapon_Shotgun_02` |
| `NPCData Stand` | `STANDING NPC` | `DA_NPC_Stand` |
| `NPCData Roaming` | `ROAMING NPC` | `DA_NPC_Roaming` |
| `NPCData Melee` | `MELEE NPC` | `DA_NPC_Melee_Roaming` |
| `Zombie Data` | `INFECTED` | `DA_Zombie_Roaming` |

So a new item of yours is one entry in the right array, and a new gun is one entry in `Weapon Classes`. The four AI fields take a Data Asset, not a character class, which means you can point the spawn rows at your own AI setups without adding rows. See [Add an item](../inventory/add_an_item.md), [Add a new gun](../weapons/add_a_new_gun.md) and [The AI settings Data Asset](../ai/ai_settings_data_asset.md).

---

## Add your own action

Say you want a row that empties every weapon you carry.

1. Open `BP_DevDebugHUD` and go to the `Build rows` function.
2. Add a call to `Add row` at the point in the list where you want it, in the category you want. Give it an id of your own, a label and a one line description.
3. In `Run action`, add a branch that tests for your id and wire what should happen.
4. Call `Set status` at the end, with the text the menu should print once it ran.

Compile and save. The row is there the next time you open the menu.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
