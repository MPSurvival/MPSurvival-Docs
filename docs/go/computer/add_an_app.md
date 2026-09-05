# Write your own app

An app is one Data Asset and one Widget Blueprint. The computer never learns its name.

---

## The recipe

1. Create a Widget Blueprint whose **parent is `BP_AppWidget`**, in `Blueprints/Widgets/World/`. Call it `BP_<Name>AppWidget`.
2. Override **`BindApp`** and build your list there.
3. Create a `DA_App_<Name>` in `Blueprints/DataAssets/Apps/Childs/` from `BP_AppDataAsset`.
4. Fill in `DisplayName`, `Icon`, `TitleColor` and `AppWidgetClass`.
5. Add the asset to `Apps` on `BP_Computer`, in the Details panel of the computer in your level.

That is all. The desktop icon, the window, the title bar, the balance readout and the close button come from the chrome.

---

## If your app needs settings

Subclass the Data Asset rather than adding fields to the base.

`BP_MarketAppDataAsset` is the example: it is a child of `BP_AppDataAsset` and adds one field, `AvailableProducts`. `DA_App_Market` is an instance of that child.

The same pattern gives you `BP_StructureAppDataAsset`, `BP_HiringAppDataAsset` and `BP_AdsAppDataAsset`. Nothing goes on the base class that only one app would read.

---

## The two functions to know

| Function | Contract |
|---|---|
| `SetApp(NewApp)` | Called by the computer. It stores the asset and calls `BindApp`. Do not override it |
| `BindApp` | Empty in the base. Override it. Clear your rows and rebuild them |
| `RefreshBalance` | Called when the balance changes. Update affordability without rebuilding |

`BindApp` has to be safe to call more than once, because it is called every time the app is opened. Clear before you fill.

Do not rebuild the list from `RefreshBalance`. Walk your existing rows and update them, the way the Structures app does with `SetAffordable`.

---

## Getting at the game

Every app resolves what it needs at the top of `BindApp`, from the game state:

```
Get Game State  →  Cast To BP_StoreGameState  →  Get Component By Class
```

That gives you `BP_StoreManager` for money and the clock, `BP_ProgressionManager` for unlock filtering, `BP_EconomyManager` for wholesale prices, `BP_StaffManager` for the roster, and so on.

Spending money goes through `BP_StoreManager.AddTransaction` with a reason from `E_TransactionReason`, and nothing else. It is the only writer of the balance in the whole project, which is what makes the daily report trustworthy.

---

## Hiding entries behind progression

If your app lists things that should be unlocked over time, filter them:

```
For Each item
    Branch  →  ProgressionManager.IsUnlocked(item)
        True  →  make the row
```

Filter **inside** the loop with a `Branch`. A function returning a filtered array does not compile here, because Blueprint arrays are invariant and an array of `PrimaryDataAsset` will not connect to a loop typed to your own asset class.

Anything not mentioned in any unlock's `Rewards` is considered unlocked, so this costs nothing until you actually write an unlock for it.

---

## Layout and style

Follow the shipped apps rather than inventing a look. They are all on the same grid, and the point of the chrome is that six apps read as one machine.

| Rule | Value |
|---|---|
| Canvas | 1920 × 1080 |
| App area | 960 tall |
| Grid | 8 px |
| Type | D-DIN, at the sizes the other apps use |
| Text colour | The `Ink` token, full opacity for data and 0.55 for labels |
| Background | The `PanelSlate` token |
| Accent | Your `TitleColor`, on the title bar and the desktop icon, and nowhere else |

Pick your title colour from the same family as the others. They all sit at S 0.57 and V 0.60 in HSV, with only the hue moving: cold hues are catalogues you buy from, warm hues are things you already own and manage. The green band is off limits, because it belongs to the currency symbol and money appears on nearly every screen.

Two things that apply to every button you add:

- Turn **Is Focusable** off, or `Tab` starts walking a focus rectangle around your app.
- Never add a tooltip. There are none anywhere in the project. A tooltip needs a hover and a wait, and it hides information the screen should have shown. If a piece of text is worth reading, give it a column.

---

## Currency

Every currency symbol in the template is a **separate text block** in `MoneyGreen`, next to the number. Not part of the same string.

That is a template-wide rule, and it applies to the HUD, price labels, the till, the terminal and every app.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
