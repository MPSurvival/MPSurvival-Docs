# Write your own app

An app is one Data Asset and one Widget Blueprint.

---

## The recipe

1. Create a Widget Blueprint whose **parent is `BP_AppWidget`**, in `Blueprints/Widgets/World/`. Call it `BP_<Name>AppWidget`.
2. Override **`BindApp`** and build your list there.
3. Create a `DA_App_<Name>` in `Blueprints/DataAssets/Apps/Childs/` from `BP_AppDataAsset`.
4. Fill in `DisplayName`, `Icon`, `TitleColor` and `AppWidgetClass`.
5. Add the asset to `Apps` on `BP_Computer`, in the Details panel of the computer in your level.

That is all. The desktop icon, the window, the title bar, the balance readout and the close button come from the chrome.

---

## The two functions to override

| Function | What to do with it |
|---|---|
| `BindApp` | Clear your rows and rebuild them. Called every time the app is opened, so it must be safe to call twice |
| `RefreshBalance` | Called when the balance changes. Update what each row can afford, without rebuilding the list |

Do not rebuild from `RefreshBalance`: walk your existing rows and update them, as the Structures app does with `SetAffordable`. Rebuilding destroys the row the player has just clicked.

`SetApp` is called by the computer and stores the asset before calling `BindApp`. Leave it alone.

---

## If your app needs settings of its own

Subclass the Data Asset rather than adding fields to the base. `BP_MarketAppDataAsset` is the example: a child of `BP_AppDataAsset` that adds `AvailableProducts`, with `DA_App_Market` as its instance. The Structures, Hiring and Ads apps do the same.

---

## Getting at the game

Every app resolves what it needs at the top of `BindApp`, from the game state:

```
Get Game State  →  Cast To BP_StoreGameState  →  Get Component By Class
```

That gives you `BP_StoreManager` for money and the clock, `BP_ProgressionManager` for unlock filtering, `BP_EconomyManager` for wholesale prices, `BP_StaffManager` for the roster.

Money moves through `BP_StoreManager.AddTransaction`, with a reason from `E_TransactionReason`. See [Money and the store rating](../store/money_and_rating.md).

---

## Hiding entries behind progression

If your app lists things that should be unlocked over time, filter them:

```
For Each item
    Branch  →  ProgressionManager.IsUnlocked(item)
        True  →  make the row
```

Filter **inside** the loop with a `Branch`. A function returning a filtered array does not compile here: Blueprint arrays are invariant, and an array of `PrimaryDataAsset` will not connect to a loop typed to your own asset class.

Anything not mentioned in any unlock's `Rewards` counts as unlocked, so this costs nothing until you write an unlock for it.

---

## Layout and style

Follow the shipped apps rather than inventing a look: six apps have to read as one machine.

| Rule | Value |
|---|---|
| Canvas | 1920 × 1080 |
| App area | 960 tall |
| Grid | 8 px |
| Type | D-DIN, at the sizes the other apps use |
| Text colour | The `Ink` token, full opacity for data and 0.55 for labels |
| Background | The `PanelSlate` token |
| Accent | Your `TitleColor`, on the title bar and the desktop icon, nowhere else |

Pick your title colour from the same family as the others: they all sit at S 0.57 and V 0.60 in HSV, with only the hue moving. Cold hues are catalogues you buy from, warm hues are things you already own and manage. Leave the green band alone — it belongs to the currency symbol, which appears on nearly every screen.

For every button you add:

- Turn **Is Focusable** off, or `Tab` starts walking a focus rectangle around your app.
- **Never add a tooltip.** There are none anywhere in the project. A tooltip needs a hover and a wait, and it hides information the screen should have shown. If a piece of text is worth reading, give it a column.

---

## Currency

Every currency symbol in the template is a **separate text block** in `MoneyGreen`, next to the number, never part of the same string. That holds on the HUD, price labels, the till, the terminal and every app.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
