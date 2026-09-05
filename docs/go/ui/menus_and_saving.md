# Menus, pause and saving

The menu is New Game, Continue, Load and Quit, and the pause menu adds Resume and Save.

---

## The screens

| Widget | What it is |
|---|---|
| `BP_MainMenuWidget` | Title, subtitle, version, and the entry list down the left |
| `BP_PauseMenuWidget` | The same layout without the subtitle, over a dimmed backdrop |
| `BP_MenuEntryWidget` | One line, 384 x 56 |
| `BP_SlotPickerWidget` | The slot list, used for new, load and save |
| `BP_SaveSlotWidget` | One slot row, with its thumbnail |

A menu is a complete Widget Blueprint, not a page built from data. **To add an entry**, drag a `BP_MenuEntryWidget` into the Designer, fill in its `Id` and `Label`, and add a branch for that `Id` in the controller.

!!! warning
    An entry's identity is its `Id`, a `Name`, not its label. An entry with an empty `Id` does nothing when clicked, silently. It is the first thing to check when a menu button feels dead.

---

## The slot picker

One widget serves three jobs, chosen by `Mode`:

| Mode | What clicking a slot does |
|---|---|
| `NewGame` | Wipes that slot and starts a fresh game |
| `LoadGame` | Loads it |
| `SaveGame` | Saves over it |

`SlotCount` is five, on the game instance. A row shows the thumbnail, `SLOT N`, the day and balance, and when it was written. An empty slot says `EMPTY`, and the one you are playing says `CURRENT`.

`Continue` loads the most recently written slot, and hides itself when there is none.

---

## What is saved

- Money, the journal, the rating, the day, the hour.
- Shelf contents, row by row.
- Placed structures, matched back by their `DA_` and transform.
- Boxes and what is in them.
- The roster and its shifts.
- Pending orders, market quotes, running campaigns, granted unlocks.
- **Customers**, because their baskets hold stock that has already left your shelves.

Not saved: any AI's current target or task, employee positions (the roster puts them back), your own position (you return to the `PlayerStart`), and a sale in progress, which is cancelled with its goods returned.

---

## Adding your own state to the save

Every manager implements two functions, and the game instance calls them:

| Function | What to do with it |
|---|---|
| `CollectInto(Save)` | Write your own state into the payload |
| `RestoreFromSave()` | Read your own state back. Call it last in your `BeginPlay` |

Add your fields to `SG_StoreSave`, then fill them in those two functions. Nothing else needs to change: the world always starts as a new game and the save overwrites it afterwards, so there is no start-up order to respect.

`SG_SlotHeader` is the second, tiny object per slot - day, balance, rating, timestamp - and it is all the menu reads to draw a row.

---

## The thumbnail

It is a real `.png` written next to the save, captured from your point of view at save time and loaded back when the menu draws the row.

Deleting a slot leaves its `.png` behind, harmlessly: it is only loaded when a header exists, and the next save to that slot overwrites it.

---

## Pause

Pause lives in `BP_StorePlayerController`, and it refuses while the closing modal or the daily report is up.

If you extend it, remember that **world timers do not run while the game is paused**. Anything a paused screen depends on has to be driven from a widget `Tick`, which does run.

---

## The menu map

`L_MainMenu` is a copy of the street with the gameplay taken out and a slowly drifting camera. Two things to watch if you build your own:

- A child of `CameraActor` **constrains the aspect ratio by default**, which puts black bars either side of the menu. Turn it off on the class defaults **and** on the placed instance.
- `UI Only` input mode survives `OpenLevel`, so the gameplay controller has to re-assert `Game Only` at `BeginPlay` or you spawn in the shop unable to move.

---

## Sounds

Every button plays the same two sounds, and they live in the **button style**, not in a graph:

| Sound | Where |
|---|---|
| `SC_UIHover` | `Style -> Hovered Sound` |
| `SC_UIClick` | `Style -> Pressed Sound` |

Those are the only two slots Slate actually plays. Keep replacements short and low: a high tonal beep reads as science fiction, which is not what a shop sounds like.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
