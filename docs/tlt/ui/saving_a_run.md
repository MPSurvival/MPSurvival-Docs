# Save and load a game

The template saves a run: what the player carries, what they learned, what they upgraded, how much health they have left, and where they are standing. It reloads all of it in one call.

Three assets do the work:

| Asset | What it is |
|---|---|
| `BP_TLTGameInstance` | The object that survives a level change. It owns saving and loading. |
| `SG_TLTGameSave` | The save itself. One of these holds a whole run. |
| `SG_MenuProfileSlot` | A small header per slot: the level name and the date, so the load screen can list slots without opening them. |

They are in `Content/TheLastTemplate/Blueprints/Misc/`.

---

## What a save contains

`SG_TLTGameSave` holds the player payload and the world state side by side.

| Field | What it remembers |
|---|---|
| `Inventory Slots` | Every item and how many of each. |
| `Known Recipes` | The recipes the player has learned. |
| `Collected Collectibles` and `New Collectibles` | The catalogue, including which entries still carry the new badge. |
| `Skill Progress` | Which levels are unlocked and which are bought. |
| `Weapons Rows` | The guns the player owns. |
| `Ammo Inventory` and `Current Ammo Weapons` | Reserve ammo per type, and the rounds sitting in each magazine. |
| `Upgrade Loadouts` | Which upgrades are fitted on which gun. |
| `Throwable Slots` | The primary and secondary throwable and their counts. |
| `Has Backpack` and `Has Light` | Whether the player picked up the bag and the flashlight. |
| `Vitals` | Health and stamina. |
| `Player Transform` | Where the player is standing and which way they face. |
| `Current Level Name` | The level to reopen on load. |
| `Level States` | One entry per level, holding what changed in the world. See [Make your level objects remember what happened](level_state_saveable_actors.md). |

Two things are deliberately outside the save. Enemies are not saved, so a level reloads with its AI as you placed it. Actors that are not saveable come back exactly as you built them.

---

## Two files per slot

A slot is written as two save objects, not one:

- `TLT_Save_0` is the header, an `SG_MenuProfileSlot` with the level name and the date;
- `TLT_Save_0_Data` is the payload, the `SG_TLTGameSave` above.

The load screen reads the headers only. Listing five slots costs five small reads instead of five whole runs.

The prefix is a variable, `Save Slot Prefix` on `BP_TLTGameInstance`. Change it there and every key follows.

---

## When the game saves

Two triggers ship.

**The pause menu.** The save entry calls `Save Checkpoint` on the game instance. That writes the payload and the header for the current slot.

**A save zone.** `BP_SaveZone` is a trigger volume you drop in a level. When the player walks in, it saves. It has one field:

| Field | What it does |
|---|---|
| `Save Once` | When true, the zone saves the first time the player enters and never again. Leave it false for a zone the player passes through often. |

To place one: drag `BP_SaveZone` into the level, resize its box the way you resize any trigger, and set `Save Once`. There is nothing to connect.

If you want a third trigger, for example an autosave when the player finishes a fight, call `Save Checkpoint` on the game instance from your own graph. That is the whole interface.

---

## The saving indicator

`BP_SaveIndicatorWidget` shows a small spinner in the bottom right while a save is being written. It listens to the `On Save Started` dispatcher on the game instance, so it costs nothing when no save is happening.

| Field | What it does |
|---|---|
| `Hold Seconds` | How long the indicator stays up at minimum. A save is usually faster than the eye, so without this you would see a flicker. |
| `Fade Speed` | How quickly it fades out afterwards. |
| `Save Sound` | The sound played when a save starts. |

!!! warning
    `Fade Speed` at `0` never finishes the fade, so the indicator stays on screen. Any value above zero is fine.

---

## Loading

`Load Game State` reads the header and the payload for a slot, reopens the level named in the save, and hands the payload back once the player exists.

The order things are restored in is fixed, and it matters: inventory, then skills, then weapons, then upgrades, then throwables, then the backpack and light, then vitals, then the transform. Upgrades have to land on guns that are already there, and equipping the backpack refreshes where the holstered weapons attach.

You do not call the pieces yourself. `Load Game State` is the whole thing.

---

## Add your own system to the save

Everything the player carries is captured through one pair of functions on `BP_PlayerCharacter`: `Capture To Save` and `Restore From Save`. The game instance calls those two and nothing else, so it never has to know what components the player has.

To add a system of your own:

1. On your component, write a function that copies its state into the save object.
2. Write the matching one that reads it back.
3. Add a field to `SG_TLTGameSave` for it.
4. Call your capture function from `Capture To Save` on the player, and your restore function from `Restore From Save`, in the position that makes sense for the order above.

That is four steps and none of them is inside the save system.

---

## Settings are a separate save

Options are not part of a run. Volume, resolution, key bindings and brightness live in `SG_MenuSettings`, written whenever the player changes a setting and read at boot. Deleting a save slot does not reset the options, and resetting the options does not touch a slot.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
