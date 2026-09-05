# Carrying, placing and throwing

There is one way to carry something in this template, and it is a physics grab. The object you are holding is a real simulated body pulled toward your hands by a `PhysicsHandleComponent`, not a mesh welded to the camera.

That is why a box you are carrying bumps into door frames, rests on a counter when you lower it, and swings a little when you turn. It is also why the code that does it is short.

---

## Making an actor carryable

Two things, and neither of them is a parent class:

1. Implement **`BPI_Carryable`**.
2. Set **Can Character Step Up On = No** on its collision body.

The second one is not optional. Without it, a box on the floor becomes the character's movement base: the movement component presses down on it, the simulated body squirts out from under your feet, and you get launched across the room.

The interface has three entries:

| Function | Returns | What it is for |
|---|---|---|
| `GetCarryBody` | `Static Mesh Component` | Which body the physics handle grabs |
| `GetCarryOffset` | `Transform` | How the object sits in your hands |
| `PickUp(Interactor)` | | What happens when it is picked up |

The object decides its own pose. `GetCarryOffset` is on the object rather than on the player because only the object knows how it wants to be held.

To make it pickable with `E`, give it a `BP_InteractableComponent` and wire its `OnInteracted` to `PickUp`, which casts the interactor to `BP_StoreCharacter` and calls `CarryObject(self)`. That is the whole hookup, and it is the same shape as every other interaction in the project.

---

## The keys

| Key | What it does |
|---|---|
| `E` | Pick up |
| `G` | Put it down where you are looking |
| `R` | Throw it |

`G` is a place, not a drop. It traces where you are aiming, up to the interaction range of 220 cm, and sets the object down with its **base** on that point, killing its linear and angular velocity. Aim at a table and the box lands on the table. Aim at nothing and it lands at the end of the trace.

`R` is a place followed by an impulse, at `ThrowSpeed`, 450 cm/s.

Carrying slows you down. `CarrySpeedScale` composes with sprint in `RefreshWalkSpeed`, which is the only place walk speed is ever written.

---

## The panel next to your hands

While you are carrying something, a small panel of key prompts is attached beside the object. The player fills it first with *Put down* and *Throw*, then hands it to the object, which adds its own lines.

A box adds *Open* / *Close*, and adds *Stock* and *Take back* only when the shelf you are looking at will actually accept the gesture. Look at a rack with a free slot and *Store* appears; look at a full one and it does not.

Each line reads its letter from the `InputAction` it is bound to, so remapping a key updates the panel with no work.

The panel is **attached** to the object rather than positioned every frame, which is why it does not jitter when you walk. Its height is measured once, in the object's own space, from the carry body's local bounds.

---

## Going through a doorway

Push a carried object into a wall and the physics handle cannot reach its target. Left alone, the body would grind against the wall forever.

When the gap between the body and the handle's target passes `GhostDistanceCm` (150 cm), the object stops colliding with the world and takes on a pale tint. Below `SolidDistanceCm` (20 cm) it goes solid again. The two thresholds are far apart on purpose, so it does not flicker while the object trails behind you.

Both values are on `BP_CarryComponent`, in `Settings|Carry`. The tint is a parameter on `M_PropsMaster`, in the `Ghost` group:

| Parameter | What it does |
|---|---|
| `GhostAmount` | Driven by the component, 0 or 1 |
| `GhostColor` | The colour it fades toward |
| `GhostFade` | How far it fades. `0.4` washes the box out without whitening it |

A material without a `GhostAmount` parameter simply ignores the call, so your own props do not have to opt in.

Releasing the object always makes it solid again, whichever way you release it.

---

## Two things it does not do

- **The object is not attached to a hand bone.** It is a body on a spring. If you want it welded, you are changing the whole component, not a setting.
- **The object follows your yaw only.** Look up or down and the box stays level. That is the right behaviour for something held in two hands, and it is a `MakeRotator(0, 0, camera yaw)` in `AdvanceCarryPose` if you want it otherwise.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
