# How interaction works

Everything the player walks up to and uses is the same actor. Doors, the gate button, the workbench and every pickup in the template are children of one class:

`Content/TheLastTemplate/Blueprints/Environments/Interactable/BP_InteractableOverlapBase`

Open it once and every interactable in the project reads the same way. This page is the mental model. The pages after it are the recipes.

---

## What one interactable is made of

Two spheres and a floating prompt.

- `Interact Zone` and `Interact Action Zone` are the two spheres. Their radii come from the two range fields below, so you never set a radius on the component itself.
- The outer radius is `Show Interact Zone Range`. Step inside it and the object becomes a candidate for the prompt.
- The inner radius is `Interact Zone Range`. Inside it, pressing the key actually does something.
- `Interact Widget` is a Widget Component carrying `BP_InteractWidget`, the prompt the player reads.

| Field | What it does | Shipped default |
|---|---|---|
| `Interact Text` | The label on the prompt | empty |
| `Error Text` | Shown in place of the label when the object refuses | `Full` |
| `Interact Zone Range` | Radius in cm inside which the key works | `60` |
| `Show Interact Zone Range` | Radius in cm at which the prompt appears | `300` |
| `Use Icon` | Show an icon instead of the label | `false` |
| `Interact Icon` | The texture used when `Use Icon` is on | empty |

The prompt itself has four states, listed in `E_InteractDisplayState`: `Hidden`, `Far`, `Near` and `NearError`. The actor moves between them on its own. You never set a state by hand, you only set the two ranges and the text.

---

## Only one prompt on screen at a time

Entering the outer sphere does not show a prompt by itself. The interactable registers on the player character, which keeps the list of candidates and elects a single winner.

To win, a candidate has to pass three tests:

1. its prompt has to project inside the viewport,
2. it has to be roughly in front of the camera, not off to the side,
3. a trace from the camera to the prompt has to reach it, so an object behind a wall is out.

Of the candidates that pass, the one closest to the player wins. When the winner changes, the player broadcasts it and every other candidate hides its prompt.

The key press is arbitrated the same way. The player broadcasts the press, and an interactable acts only if it is the active one and that press has not been handled already. Two pickups lying on top of each other cannot both answer it.

This is why a crowded shelf still reads clearly, and it is also why a large `Show Interact Zone Range` is not a problem: only the elected candidate draws anything.

---

## Icon, text, and a count

`Use Icon` chooses between an icon and a label. They are alternatives, not additive: with `Use Icon` on, the prompt draws `Interact Icon` and no label, with it off it draws `Interact Text`.

Pickups add `Show Amount`. With it on, the prompt draws how many the pile holds next to the label, and the pickup base rebuilds that label through `Refresh Amount` whenever the number changes. A child says what the number is by overriding `Get Amount`, which is how an item pile of eight parts and a box of eight pistol rounds share the same prompt.

---

## Refusing an interaction, and saying why

`Is Errored` is a function on the base, made to be overridden on a child. It returns two things: whether the object refuses right now, and the reason to show.

When it returns true and the player is inside the inner sphere, the prompt goes to `NearError` and draws that reason instead of the label. The shipped default in `Error Text` is `Full`.

Your own reasons go in the same place. A door that needs a key, a machine with no power, a pile you have no room for: each one is a single override of `Is Errored` on the child, and none of them touch the base.

---

## The one event a child fills in

The base declares `Interact Action`. That is where a child does its work, and in the shipped classes it is usually the only event they implement. The gate button moves its gate there, the door starts its timeline, the item pickup pushes itself into the bag.

So a brand new interactable of your own is: a child of `BP_InteractableOverlapBase`, a mesh, an `Interact Text`, an `Interact Action` event, and an `Is Errored` override if it can ever refuse.

## Pickups sit on a second base class

Anything the player picks up goes through `BP_Interactable_PickupBase`, a child of the interaction base that adds the mesh handling.

| Field | What it does | Shipped default |
|---|---|---|
| `Static Mesh` | The mesh, pushed into the pickup's mesh component at construction | `EditorCube` |
| `Use Physics` | Let the object fall and settle instead of floating | `true` |
| `Use Overlay Material` | Draw the highlight material over the mesh | `true` |
| `Show Amount` | Draw the count on the prompt | `false` |

It also tightens the two ranges, to `100` and `200`, because a pickup on the floor should not claim the prompt from across the room.

If a pickup renders as a plain engine cube in your level, it is that `EditorCube` default: `Static Mesh` was never set.

Items, ammo, weapons, throwables, books, the light and the backpack are all children of this one class.

---

## What ends up in a save

`BP_Interactable_DoorBase` and `BP_Interactable_PickupBase` implement `BPI_Saveable`, the interface the level state save calls: `Get Save Id`, `Capture State`, `Restore State`. A pickup captures where it is and whether it is still there. A door captures its own state.

Nothing else on this page is saved.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
