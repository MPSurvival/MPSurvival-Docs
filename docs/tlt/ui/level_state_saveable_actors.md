# Make your level objects remember what happened

A pickup the player took stays taken. A door the player opened stays open. Both work because those actors implement one interface, `BPI_Saveable`, and nothing else.

You can do the same on any actor of your own. You never have to open the save system to do it.

- The interface: `Content/TheLastTemplate/Blueprints/Interfaces/BPI_Saveable`
- The struct it fills: `Content/TheLastTemplate/Blueprints/Structures/Save/S_SavedActorState`

---

## What per level state does

When the game saves, `BP_TLTGameInstance` collects every actor in the level that implements `BPI_Saveable`, asks each one for its state, and stores the whole list under the name of the current level. The lists live in `Level States` on `SG_TLTGameSave`, one entry per level. When a save is loaded, each state is handed back to the actor it came from.

Three things it does not do:

- It does not spawn anything. Only actors you already placed in the level are restored.
- It does not touch a level that has no entry in the save. A level the player never reached comes up exactly as you built it.
- It does not carry the player. Inventory, weapons, vitals and the rest are a separate part of the save, described in [Save and load a game](saving_a_run.md).

---

## The three functions

`BPI_Saveable` has three members, all in the `Saveable` category.

| Member | Kind | What it is for |
|---|---|---|
| `Get Save Id` | Function, returns `Id` | The name that ties this actor to its saved entry. It has to be the same value every session. |
| `Capture State` | Function, returns `State` | Build one `S_SavedActorState` describing the actor right now. |
| `Restore State` | Event, takes `State` | Put that state back on the actor. |

`Get Save Id` and `Capture State` return a value, so they show up as functions to implement. `Restore State` returns nothing, so it is an **event** you add in the Event Graph. Looking for it in the function list is the first thing that wastes ten minutes.

`S_SavedActorState` is what you fill and what you get back:

| Field | Type | What to put in it |
|---|---|---|
| `Id` | Name | The same value your `Get Save Id` returns. This is how an entry finds its actor again. |
| `Destroyed` | Boolean | A flag for an actor that should not come back. Neither shipped implementation sets it, see below. |
| `Transform` | Transform | Where the actor is. Fill it if the actor can move, leave it if it cannot. |
| `Data` | Map | Anything else the actor needs to remember. Keys and values are text, so a boolean or a number goes in converted and comes back out converted. |

---

## Make one of your actors saveable

Say you built a generator the player can switch on.

1. Open the Blueprint. **Class Settings**, then under **Interfaces** click **Add** and pick `BPI_Saveable`. Compile.
2. In **My Blueprint**, under **Interfaces**, double click `Get Save Id`. Add a **Get Object Name** node with **Self** in its **Object** pin and connect it to the `Id` return. That is what the shipped pickups and doors do, and it gives a stable name to an actor placed in a level.
3. Double click `Capture State`. Add a **Make S Saved Actor State** node and wire it to the `State` return.
4. On that node, call your own `Get Save Id` and feed the result into `Id`. Feed `Get Transform` into `Transform` if the actor moves.
5. Build a map for `Data` with one entry per value you care about, for example the key `Running` and the value `true`.
6. In the **Event Graph**, right click and add **Event Restore State**. Break the `State` pin, read your keys back out of `Data` with **Find**, and apply them.
7. Compile and save.

That is all. The generator is now part of every save written from then on.

!!! warning
    The id is the actor's own name. Renaming the actor in the outliner after a save exists breaks the link: the saved entry no longer matches anything, the actor counts as missing, and it is destroyed on load. Rename before you start testing, not after.

---

## Reading and writing your own values

`Data` is the only field you are free to shape. The shipped door uses it for exactly this: it writes two entries, one for open or closed and one for locked, converts the booleans to text on the way in, and compares the text on the way out.

Keep the keys short and keep them constant. A key you rename is a value your actor will not find again in an older save, and **Find** returns the empty value without complaining.

---

## What already saves itself

Two classes implement the interface, and everything else inherits from them.

| Class | What it remembers |
|---|---|
| `BP_Interactable_PickupBase` | Its transform, so a physics pickup that rolled under a shelf comes back under the shelf. |
| `BP_Interactable_DoorBase` | Open or closed and locked or not, applied without replaying the opening animation. |

Every item, ammo box, throwable, weapon, book and collectible pickup derives from `BP_Interactable_PickupBase`, and all three door variants derive from `BP_Interactable_DoorBase`. You do not have to do anything to them.

Two placeable actors are not covered, on purpose, because they are demo pieces: `BP_Interactable_GateButton` and `BP_Interactable_Workbench`. A gate the player opened is closed again after a load. If you want either to persist, implement the interface on it the same way.

One gap worth knowing: a pickup does not remember a partial take. `Amount` lives on `BP_Interactable_Pickup_Item` and is not part of the captured state, so a pile the player took half of is either still there whole or gone.

---

## An actor that is missing is destroyed

An actor that implements the interface but has no matching entry in the loaded list is destroyed.

That sounds harsh and it is the point. A pickup the player collected was already destroyed when the save ran, so it could not report itself, and its absence from the list is the only record that it was taken. Restoring the level means deleting it again.

The consequence for you: an actor that fails to report itself disappears. If a saveable actor of yours vanishes after a load, the id it returns is not the id that was saved.

Nothing happens to a level with no entry at all. Missing means missing from a list that exists, not missing a list.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
