# Doors the Warden can open

A sliding door opens on its own when an enemy walks into it. The door decides, not the enemy: the Warden has nothing to set up, and neither does an enemy you make yourself. By the end of this page your own door, hatch or gate opens for enemies too.

- The component on the doors: `Content/MPFriendslop/Blueprints/ActorComponents/BP_AIDoorOpenerComponent`
- The shipped doors: `Content/MPFriendslop/Blueprints/Environments/Doors/`

In `L_Procedural`, every doorway between two modules gets a `BP_SlidingDoor_Single`, and the vault module has a `BP_SlidingDoor_Double` worked by two levers.

---

## How it works

The Warden's paths go straight through doorways, closed or not: the moving parts of the doors do not count for the nav mesh. When a pawn walks into a door, the door's `BP_AIDoorOpenerComponent` checks who it is, on the server. A pawn possessed by an `AIController` fires `On AI Pawn Bumped`. A player, a thrown crate or a wall does nothing.

Each door answers `On AI Pawn Bumped` with its own way of opening.

| Door | What happens when an enemy bumps it |
|---|---|
| `BP_SlidingDoor_Single` | Slides open, unless a player is holding it |
| `BP_SlidingDoor_Double` | Opens, as if its lever were pulled |
| `BP_ExtractionTrap`, levers, buttons | Nothing. They have no `BP_AIDoorOpenerComponent` |

The enemy touches the door and waits while the door slides out of its way, under a second at the default `Drive Speed`, then walks on. It opens doors on its patrol, while it searches and while it chases you. It never closes one.

A player who grabs a single door and holds it keeps it shut: the Warden never forces a held door. It keeps pushing, and the door opens as soon as the player lets go.

A door does not stop a shot. Once a door is open, even partly, the Warden sees and fires through the gap as it would anywhere else.

---

## Settings

On `BP_AIDoorOpenerComponent`, in the door's **Components** panel:

| Field | What it does | Default |
|---|---|---|
| `Opens For AI` | Untick it on a placed door to keep enemies out, for example a locked vault | ticked |

On `BP_GrabAxisComponent` of a single door:

| Field | What it does | Default |
|---|---|---|
| `Drive Speed` | How fast the door opens by itself, in full strokes per second | `1`, about one second |

---

## Make your own door open for enemies

1. Open your door and add a `BP_AIDoorOpenerComponent` in the **Components** panel.
2. Select it. In the **Details** panel, under **Events**, click the **+** next to `On AI Pawn Bumped`.
3. From that event, open the door the way it already opens, for example by setting its replicated state. `Pawn` is the enemy that bumped it.
4. Select each moving part in the **Components** panel and untick `Can Ever Affect Navigation`.
5. In **Class Defaults**, tick `Replicates`.

`On AI Pawn Bumped` only fires on the server, so the state you set there reaches every player the usual way. A player who joins late sees the door as it is.

It fires again on every frame the enemy keeps pushing. Open the door in a way that does no harm when it runs twice, like setting `Is Open` to true.

---

## The start room is a safe zone

The Warden never walks into the start room of `L_Procedural`, but it shoots into it when it sees you. A `NavModifierVolume` with the `NavArea_Null` area covers the inside of the room and removes the nav mesh there. The Warden walks up to the doorway and stops.

To make another safe room, place a `NavModifierVolume` over it and set `Area Class` to `NavArea_Null`. Keep the doorway itself outside the volume.

---

## Mistakes that cost time

- **The Warden stops in front of every door.** A moving part still has `Can Ever Affect Navigation` ticked. The closed door cuts the nav mesh, so no path goes through it. Press `P` in the viewport during play: the green floor should run through the doorway.
- **The door opens on the host only.** `Replicates` is not ticked on your door.
- **The Warden walks into the door and nothing happens.** The moving part has no collision, or its collision does not block `Pawn`. No hit, no event.
- **The Warden stops in the doorway, its head against the top.** The frame of a single door is clear to `220` cm across its full width, and `250` cm in the middle only. The Warden is `220` cm tall. A taller enemy needs a taller frame, and a taller `Agent Height` on the nav mesh so its paths avoid what it cannot fit under.
- **Players open the door by walking into it.** That is not this component: it only reacts to pawns possessed by an `AIController`. Look for another event on your door.

!!! warning
    A `BP_AIDoorOpenerComponent` on the extraction trap, or on a parent the trap inherits from, lets the Warden open it just by walking into it. There is no error message. Add the component to the doors themselves, never to `BP_DoorBase`.

---

Next: [Make an enemy variant or your own enemy](make_your_own_enemy.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
