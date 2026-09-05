# Menus, pause and saving

There is no options screen. The menu is New Game, Continue, Load and Quit, and the pause menu adds Resume and Save.

---

## The screens

| Widget | What it is |
|---|---|
| `BP_MainMenuWidget` | Title, subtitle, version, and the entry list down the left |
| `BP_PauseMenuWidget` | The same layout without the subtitle, over a dimmed backdrop |
| `BP_MenuEntryWidget` | One line, 384 × 56 |
| `BP_SlotPickerWidget` | The slot list, used for all three of new, load and save |
| `BP_SaveSlotWidget` | One slot row, with its thumbnail |

A menu screen is a **complete Widget Blueprint**, not a page built from data. Its entries are `BP_MenuEntryWidget` instances dropped in the Designer with their `Id` and `Label` filled in.

Adding a screen means duplicating a widget. Adding an entry means dragging a row in and adding a branch in the controller.

**An entry's identity is its `Id`, a `Name`, not its label.** An entry placed in the Designer with an empty `Id` does nothing when clicked, silently, and only with the mouse. It is the first thing to check when a menu button feels dead.

---

## The slot picker

One widget serves three jobs, chosen by `Mode`, an `E_SlotPickerMode`:

| Mode | What clicking a slot does |
|---|---|
| `NewGame` | Wipes that slot and starts a fresh game |
| `LoadGame` | Loads it |
| `SaveGame` | Saves over it |

`SlotCount` is five, and it lives on the game instance because the Continue logic needs it too.

A row shows the thumbnail, `SLOT N`, the day and balance, and the date it was written. An empty slot says `EMPTY`. The slot you are currently playing is marked `CURRENT`.

---

## Continue

`Continue` loads the most recently written slot. `ResolveContinueSlotIndex` reads the five headers, returns the one with the newest timestamp, and returns `-1` if there are none.

It is the only answer to "which save do I resume", and the Continue entry hides itself when it returns `-1`.

---

## How saving is put together

One rule explains the whole design: **the game instance is the only thing that talks to disk, and each manager is the only thing that talks about its own state.**

`BP_StoreGameInstance` owns the slot naming convention, writing, reading, deleting and the thumbnail. Each of the six managers exposes two functions:

| Function | What it does |
|---|---|
| `CollectInto(Save)` | Writes its own state into the payload |
| `RestoreFromSave()` | Reads its own state back |

Nobody touches another manager's variables. `AddTransaction` stays the only writer of money **during play**, and `RestoreFromSave` is the only writer **at load**.

Two objects go to disk per slot:

| Object | What it holds |
|---|---|
| `SG_SlotHeader` | Day, balance, rating, timestamp. This is all the menu loads to draw a row |
| `SG_StoreSave` | The whole game |

The menu never deserialises a whole game to show you a list.

**There is no `BeginPlay` order to respect.** Every manager calls `RestoreFromSave` last in its own `BeginPlay`, after initialising itself fresh. The world always starts as a new game and the save then overwrites it. One branch, no dependency between components, and the HUD reads the right numbers because restoration happens at the same moment the old initialisation did.

---

## The thumbnail

A `SaveGame` object cannot hold a texture, so the thumbnail is a real `.png` written next to the save in `Saved/SaveGames/`.

It is rendered from your actual point of view with a `SceneCapture2D` at save time, and read back with `ImportFileAsTexture2D` when the menu draws the row.

One consequence: deleting a slot leaves its `.png` behind, because there is no Blueprint node to delete an arbitrary file. It is harmless. The thumbnail is only loaded when a header exists, and the next save to that slot overwrites it.

---

## What is saved, and what is not

Saved:

- Money, the journal, the rating, the day, the hour.
- Shelf contents, row by row.
- Placed structures, matched back by their `DA_` and transform rather than rebuilt from scratch.
- Boxes and what is in them.
- The roster and its shifts.
- Pending orders, market quotes, running campaigns, granted unlocks.
- **Customers**, because their baskets hold stock that has already left your shelves.

Not saved:

- Any AI's current target, task or held object. Those are actor references, which come back as nothing, and a behaviour tree restarts at its root anyway.
- Employee positions. `EvaluateShifts` puts them back from the roster, and an employee carries nothing that matters.
- Your position. You come back at the `PlayerStart`.
- A sale in progress. It is cancelled and the goods are returned, which is exact rather than approximate: money only moves in `FinishSale`, and that call is all or nothing.

Boxes are razed and respawned rather than reconciled, because they hold no hand-authored settings and their contents are rebuilt from the product anyway.

---

## Pause

Pause lives in `BP_StorePlayerController`, next to the store-closing flow. It refuses while the closing modal or the daily report is up.

One thing to know if you extend it: **world timers do not run while the game is paused.** Anything a paused screen depends on has to be driven from a widget `Tick`, which does run, or called directly.

---

## The menu map

`L_MainMenu` is a copy of the street with all the gameplay taken out, and a slowly drifting camera.

Two things caught out during that build, both worth knowing if you make your own menu level:

- A child of `CameraActor` **constrains the aspect ratio by default**, which puts black bars either side of the menu. Turn it off on the class defaults **and** on the placed instance.
- `UI Only` input mode survives `OpenLevel`. The store controller re-asserts `Game Only` at `BeginPlay`, otherwise you spawn in the shop unable to move.

---

## Sounds

Every button in the project plays the same two sounds, and they live in the **button style**, not in a graph.

| Sound | Where |
|---|---|
| `SC_UIHover` | `Style → Hovered Sound` |
| `SC_UIClick` | `Style → Pressed Sound` |

Only those two slots are actually played by Slate. Changing a UI sound is a field in the Details panel of a button.

They are short and low. A high, tonal beep reads as science fiction, which is not what a shop sounds like.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
