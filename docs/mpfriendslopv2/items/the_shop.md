# Add an item to the shop

A shop entry is one Data Asset that points at an inventory item, plus one line in the terminal's catalogue. No graph to open.

By the end of this page your item has a tile on the shop terminal, costs what you decide, and drops out in front of the terminal when a player orders it.

The item itself must exist first. If it does not, see [Add an inventory item](add_an_item.md).

- The shop entries: `Content/MPFriendslop/Blueprints/DataAssets/Shop/Childs/`
- The terminal: `Content/MPFriendslop/Blueprints/Environments/Fixtures/BP_ShopTerminal`

---

## How the shop works

The money is the team's extracted value: what the team has already sold at the extraction point during this run. Buying spends it, so every purchase slows the quota down. The recap shows it as a negative `SUPPLY DROP` row. See [How a run works](../loot/how_a_run_works.md). Every item in the catalogue can be bought from the start of the run: the price is the only condition.

The terminal is not used with an interaction prompt. Its screen is a real widget in the world, and the player points at it with the camera:

1. Look at a tile and press `E`. One unit goes into the cart, and the tile shows a `×N` badge.
2. Press `E` on the badge to take one back out.
3. Press `E` on `CONFIRM ORDER`.

The server checks the order, spends the money and, after `Delivery Seconds`, spawns one pickup per unit at the terminal's `DeliveryPoint`. Each delivered item arrives with its full `Max Charge`: a bought flashlight is charged, a bought shotgun is loaded.

---

## The recipe

1. In `Blueprints/DataAssets/Shop/Childs/`, duplicate `DA_Shop_Medkit`.
2. Name it `DA_Shop_<Name>`.
3. Fill in the fields below.
4. Open `BP_ShopTerminal` and add your Data Asset to `Shop Catalog`.

Step 4 is the one people forget. The server refuses any item that is not in the terminal's `Shop Catalog`, and the screen only draws what is in it.

The terminal placed in the start room of `L_Procedural`, `Start_ShopTerminal`, keeps the class value, so adding the entry in `BP_ShopTerminal` is enough. To give one terminal its own list, set `Shop Catalog` on that placed terminal instead.

Every `DA_Shop_*` is a `BP_ShopItemDataAsset`, the class in `Blueprints/DataAssets/Shop/`. To start from an empty entry instead of a copy, right click in `Childs/`, then **Miscellaneous**, then **Data Asset**, and pick `BP_ShopItemDataAsset`.

---

## The fields

All in `Settings|Shop`.

| Field | What it does | Shipped in DA_Shop_Medkit |
|---|---|---|
| `Display Name` | The name on the tile | `MEDKIT` |
| `Icon` | The picture on the tile. The shipped entries reuse their item's icon | `T_Icon_Tool_Medkit` |
| `Price` | Cost of one unit. The server computes the total from this field, never from what the screen sends | `100` |
| `Category` | The tab the tile appears on | `Medical` |
| `Delivered Item` | The `DA_Item_*` that drops out. The terminal spawns its `Pickup Class` | `DA_Item_Medkit` |

The four shipped entries:

| Entry | Price | Category |
|---|---|---|
| `DA_Shop_Flashlight` | `150` | `Tools` |
| `DA_Shop_Shotgun_SawedOff` | `250` | `Tools` |
| `DA_Shop_Medkit` | `100` | `Medical` |
| `DA_Shop_Battery` | `60` | `Utility` |

The prices are starting values. Balance them for your game.

---

## Tabs

The tabs are built from the enum `E_ShopCategory`: `Tools`, `Medical`, `Carry` and `Utility`. `Carry` ships with no item, so its tab is empty.

Add an entry to `E_ShopCategory` and the screen gets one more tab, labelled with the entry's display name. Each tab draws `Slot Count` tiles, `8` by default on `WBP_ShopTerminal`, and there is no second page.

---

## The terminal

| Field | What it does | Default |
|---|---|---|
| `Shop Catalog` | What this terminal sells | the four entries above |
| `Delivery Seconds` | Delay between the order and the drop | `10` |
| `Delivery Stack Spacing` | Distance between two delivered pickups, in cm | `60` |
| `Max Order Distance` | The server refuses an order from a player farther than this from the terminal, in cm | `200` |

Move the `DeliveryPoint` component to change where the order lands. While a delivery is on its way, the terminal refuses a second order.

To test the shop without carrying loot first, raise `Starting Extracted Value` on the game mode. See [Change the quota, the run length and the recap](../loot/tune_the_run.md).

---

## Hooking your own effects

`BP_ShopTerminal` has two dispatchers. Bind a sound, a light or a quest step to them without opening the terminal.

| Dispatcher | Fires when | Fires on |
|---|---|---|
| `On Order Result` | An order is accepted or refused. Gives `Success` and a `Fail Reason` text such as `INSUFFICIENT FUNDS` or `TOO FAR FROM TERMINAL` | owning player (the one who ordered) |
| `On Delivery Started` | An accepted order starts its countdown. Gives `ETA Seconds` | every player |

---

## On your own character

`BP_FriendslopCharacter` already has both of these. On your own pawn, add:

- `BP_ShopComponent`, which sends the order to the server
- `BP_ShopPointerComponent`, which points at the screen from the pawn's camera. `Pointer Distance` is its reach, `70` cm by default

See [How the player character works](../player/how_the_player_works.md#put-it-on-your-own-character).

!!! warning
    The pointer is stopped by anything that blocks the `Visibility` channel between the camera and the screen. That includes a world widget or a mesh attached to your own pawn. Tiles then ignore every click, with no error. Give those components no collision, as the template does on its own face and nameplate.

---

## Your own terminal or screen

The terminal and its screen only talk through two interfaces, so you can replace one half without opening the other.

`BPI_ShopTerminal` is the terminal half. `BP_ShopTerminal` implements it, and `WBP_ShopTerminal` and `BP_ShopComponent` know nothing else about the terminal.

| Function | Who calls it | What `BP_ShopTerminal` does |
|---|---|---|
| `GetShopCatalog` | the screen | Returns `Items`, its `Shop Catalog` |
| `GetShopFunds` | the screen | Returns `Funds`, the team's extracted value |
| `ConfirmOrder` | the screen, on `CONFIRM ORDER` | Passes `Order` to `RequestOrder` on the local player's `BP_ShopComponent` |
| `ServerConfirmOrder` | `BP_ShopComponent`, on the server | Checks `Order` and `Orderer`, spends, and starts the delivery |
| `ReportOrderResult` | `BP_ShopComponent`, on the ordering player's machine | Fires `On Order Result` and passes `Success` and `Fail Reason` to the screen |

`Order` is a list of `S_ShopOrderLine`, one `Item` and one `Quantity` per line. To answer from your own terminal, call `Client_ReportOrderResult` on the `Orderer`'s `BP_ShopComponent`, on the server, with your terminal as `Terminal`. It calls `ReportOrderResult` back on your terminal, on that player's machine.

`BPI_ShopScreen` is the screen half. `WBP_ShopTerminal` implements it.

| Function | When the terminal calls it | What `WBP_ShopTerminal` does |
|---|---|---|
| `SetShopTerminal` | At `BeginPlay`, with itself as `Terminal` | Keeps `Terminal`, then builds its tabs and tiles |
| `ShowOrderResult` | From `ReportOrderResult` | Empties the cart on success, shows the `Fail Reason` otherwise |
| `ShowDelivery` | When a delivery starts, with `ETA Seconds` | Shows the countdown |

`BP_ShopTerminal` finds its screen through its `ScreenWidget` component. To use your own screen, implement `BPI_ShopScreen` on your widget and set it as the `Widget Class` of `ScreenWidget`.

---

To make the bought item something the player holds and uses, see [Make a held item, its hold pose and hand IK](make_a_held_item.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
