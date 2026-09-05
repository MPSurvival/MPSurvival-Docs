# Carrying, placing and throwing

Carrying is a physics grab: the object in your hands is a simulated body pulled toward them, not a mesh welded to the camera. It bumps into door frames, rests on a counter when you lower it, and swings a little when you turn.

---

## The keys

| Key | What it does |
|---|---|
| `E` | Pick up |
| `G` | Put it down where you are looking |
| `R` | Throw it |

`G` is a place, not a drop: it traces where you are aiming, up to 220 cm, and sets the object down with its base on that point. Aim at a table and the box lands on the table. `R` does the same, then adds an impulse at `ThrowSpeed` (450 cm/s).

Carrying slows you down, by `CarrySpeedScale` on `BP_CarryComponent`.

---

## Making your own actor carryable

Two things:

1. Implement **`BPI_Carryable`**.
2. Set **Can Character Step Up On = No** on its collision body.

The second one is not optional. Without it the box becomes the character's movement base, and the simulated body squirts out from under your feet and launches you across the room.

The interface has three entries:

| Function | Returns | What it is for |
|---|---|---|
| `GetCarryBody` | `Static Mesh Component` | Which body is grabbed |
| `GetCarryOffset` | `Transform` | How the object sits in your hands |
| `PickUp(Interactor)` | | What happens when it is picked up |

To make it pickable with `E`, add a `BP_InteractableComponent` and wire its `OnInteracted` to `PickUp`, which casts the interactor to `BP_StoreCharacter` and calls `CarryObject(self)`.

---

## The panel next to your hands

While you carry something, a small panel of key prompts sits beside it. It always shows *Put down* and *Throw*, and the object adds its own lines: a box adds *Open* / *Close*, and adds *Stock* or *Take back* only when the shelf you are looking at will accept the gesture.

Each line reads its letter from the `InputAction` it is bound to, so remapping a key updates the panel with no work.

---

## Going through a doorway

Push a carried object into a wall and it would otherwise grind there. Instead, once it falls far enough behind your hands it stops colliding with the world and goes pale, then turns solid again as it catches up.

Both distances are on `BP_CarryComponent`, in `Settings|Carry`:

| Field | Shipped |
|---|---|
| `GhostDistanceCm` | `150` |
| `SolidDistanceCm` | `20` |

The pale tint comes from the `Ghost` group on `M_PropsMaster`: `GhostColor` is the colour it fades toward and `GhostFade` how far (`0.4` washes the box out without whitening it). A material without those parameters simply ignores it, so your own props do not have to opt in.

Releasing the object always makes it solid again.

---

## Two things to know about the hold

- **The object is a body on a spring, not a bone attachment.** Welding it to a hand means changing the whole component, not a setting.
- **The object follows your yaw only.** Look up or down and the box stays level. That is the right behaviour for something held in two hands, and it is a `MakeRotator(0, 0, camera yaw)` in `AdvanceCarryPose` if you want it otherwise.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
