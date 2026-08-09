# Add an object the player can inspect

You end up with an object sitting in the level that the player can hold up, turn around, read and act on. It costs one Data Asset and one actor dragged into the level. You do not open a graph.

If you want the mental model first, read [How 3D inspection works](how_inspection_works.md).

- The Data Assets: `Content/TheLastTemplate/Blueprints/DataAssets/Inspect/Childs/`
- The actor you place: `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/Inspect/BP_Interactable_Pickup_InspectableBase`

---

## Before you start

You need one static mesh imported in the project. Two things about that mesh matter:

- Its pivot should sit near the middle of the object. The inspect view turns the mesh around its own origin, so a pivot stuck in a corner makes the object swing around instead of spinning in place. Fix that in your modelling tool before you import.
- Its real size does not matter. The inspect view fits the object to the frame for you, and you tune that fit with one number, `Frame Fill`.

---

## Make the Data Asset

1. Go to `Content/TheLastTemplate/Blueprints/DataAssets/Inspect/Childs/`.
2. Right click `DA_Inspect_Note_Sorry` and pick **Duplicate**. It is the simplest of the eight, one action and nothing else. Name the copy after your object, for example `DA_Inspect_Radio`.
3. Open it and set `Mesh` to your static mesh.
4. Set `Display Name`. This is the short name the inspect view shows for the object, not a unique title. All three shipped cards use `Trading Card`.
5. Set `Inspect Rotation`. This is the angle the object is handed to you at, before the player turns it. Every shipped object uses `0 / 90 / 90`. Turn the numbers until the front of your object faces the camera.
6. Set `Frame Fill`. Raise it if the object reads too small on screen, lower it if it runs off the edges. The three books use `0.75`, the notes and the cards use `0.82`.
7. Fill in `Actions`, one row per button. See below.
8. Save.

You can also make one from scratch: right click in the Content Browser, **Miscellaneous**, **Data Asset**, and pick `BP_InspectableDataAsset` as the class. Duplicating just saves you from picking the wrong class.

### The fields

| Field | What it does |
|---|---|
| `Display Name` | The name shown for the object while it is being inspected |
| `Mesh` | The static mesh shown in the inspect view |
| `Inspect Rotation` | Starting angle of the object, before the player turns it |
| `Frame Fill` | How much of the frame the object fills. Shipped values run from `0.75` to `0.82` |
| `Actions` | Zero, one or two rows. Each row is one button |

### The action rows

Each row of `Actions` is one button on screen. The row carries `Action Class` (what the button does), `Slot` (`Primary` or `Secondary`), `Label` (the word on the button), `Closes Inspect` (whether running it leaves the inspect view), and a small bag of parameters, `Text Param`, `Text Lines`, `Float Param`, `Int Param` and `Class Param`, that the chosen action class reads or ignores.

The template ships action classes for reading a document, taking an item, taking a collectible, learning a crafting recipe and unlocking a player upgrade. Which one to pick, what to put in the parameter bag, and how to write your own are covered on the next page.

If you leave `Actions` empty, the object still works. The player can open it and turn it, and no button appears.

---

## Put it in the level

1. Drag `BP_Interactable_Pickup_InspectableBase` into the level.
2. In the Details panel, set `Static Mesh` to your mesh. This is the mesh the player sees lying in the world, and it is a different field from the `Mesh` on the Data Asset.
3. Set `Inspect Data` to the Data Asset you just made.
4. Play. Walk up to it and press the interact key.

!!! warning "Two mesh fields, both needed"
    `Static Mesh` on the actor is the object in the world. `Mesh` on the Data Asset is the object in the inspect view. Setting one does not set the other, and a missing `Mesh` gives you an empty inspect view with no error. A grey cube in the level is the other half of the same mistake: the pickup base ships with the engine's `EditorCube`, so a forgotten `Static Mesh` at least shows.

### The actor fields worth knowing

| Field | Default here | What it does |
|---|---|---|
| `Inspect Data` | `None` | The Data Asset this object opens |
| `Static Mesh` | `EditorCube` | The mesh shown in the level |
| `Use Physics` | on | Turn it off for anything you placed by hand on a shelf or a table |
| `Use Overlay Material` | on | The highlight material drawn on the object when you look at it |
| `Use Icon` | on | The base class turns this on, so the prompt is an icon, not a word |
| `Interact Icon` | `T_Journal` | The icon used while `Use Icon` is on |
| `Interact Text` | `Examine` | Shown instead of the icon when you turn `Use Icon` off. The two are alternatives, never both |
| `Show Interact Zone Range` | `200` | Distance at which the prompt appears |
| `Interact Zone Range` | `100` | Distance at which the key actually works |

---

## A child class, or an instance

Both are used in the shipped content, and the difference is only how often you place the thing.

Make a **child Blueprint** of `BP_Interactable_Pickup_InspectableBase` when you will place the same object many times or want it findable in the Content Browser. Set `Static Mesh` and `Inspect Data` in the class defaults, and every copy you drag in is already correct. The three books are done this way: `BP_Interactable_Pickup_Book_Field`, `BP_Interactable_Pickup_Book_Gunsmith` and `BP_Interactable_Pickup_Book_Medical` add nothing but those two values.

Place the **base class directly** for a one off. The two notes and the three cards have no class of their own: they are instances of `BP_Interactable_Pickup_InspectableBase` with their Data Asset set on the instance. For a note that exists once in one room, a whole Blueprint is not worth it.

---

## What ships, and what each one shows

Eight inspectable Data Assets come with the template.

| Data Asset | `Display Name` | Buttons |
|---|---|---|
| `DA_Inspect_Book_Field` | Field Guide | `Learn` on `Primary`, unlocks an upgrade and closes |
| `DA_Inspect_Book_Gunsmith` | Gunsmithing | `Learn` on `Primary`, unlocks an upgrade and closes |
| `DA_Inspect_Book_Medical` | Field Medicine | `Learn` on `Primary`, unlocks an upgrade and closes |
| `DA_Inspect_Note_Sorry` | Note | `Read` on `Primary`, one button and no reward |
| `DA_Inspect_Note_Recipe_Molotov` | Recipe | `Learn` on `Primary`, `Read` on `Secondary` |
| `DA_Inspect_Card_Comet` | Trading Card | `Read` on `Primary`, `Take` on `Secondary` |
| `DA_Inspect_Card_Hollow` | Trading Card | `Read` on `Primary`, `Take` on `Secondary` |
| `DA_Inspect_Card_Howler` | Trading Card | `Read` on `Primary`, `Take` on `Secondary` |

All eight open at `0 / 90 / 90`. The five documents take their `Mesh` from `Content/TheLastTemplate/Meshes/Documents/`, the three cards from `Content/TheLastTemplate/Meshes/Collectibles/`.

Copy the one closest to what you want. A pure story object copies `DA_Inspect_Note_Sorry`. An object that teaches something copies a book.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
