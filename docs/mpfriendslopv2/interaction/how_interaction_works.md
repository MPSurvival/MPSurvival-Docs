# How interaction works

Anything the player looks at and uses with `E` goes through one interface and one component. The interface, `BPI_Interactable`, goes on the thing you use. The component, `BP_InteractionComponent`, goes on the player.

- The interfaces: `Content/MPFriendslop/Blueprints/Interfaces/`
- The components: `Content/MPFriendslop/Blueprints/ActorComponents/`

The player character never knows what it is looking at. The object answers for itself: its prompt, how it is used, and what happens. So a new kind of interactable never needs an edit on the player.

This page is the mental model. The pages after it are the recipes.

---

## The interface on the object

`BPI_Interactable` has five functions. An actor that implements it can be used. It also needs the two settings listed after the component table.

| Function | What the object does in it |
|---|---|
| `GetInteractionData` | Returns `Prompt` (the line under the ring), `Type` and `Duration` |
| `CanInteract` | Returns true when the player passed as `Interactor` may use it right now |
| `Interact` | Does the actual work. Runs on the server |
| `StartFocus` | The player started looking at it. For a cosmetic highlight only |
| `EndFocus` | The player stopped looking at it |

`Type` is an `E_InteractionType`, with three entries:

| Type | How the player uses it |
|---|---|
| `Simple` | One press |
| `Hold` | Hold the key until the ring fills. `Duration` is the time in seconds. Looking away or stepping out of range cancels it. The player then presses again |
| `Spam` | Each press adds `Spam Press Gain` to the ring, and it drains on its own between presses. It completes when the total reaches `Duration`. If it drains to empty, the attempt is canceled |

There is no Enhanced Input trigger and no timer to add for `Hold` or `Spam`. The one `IA_Interact` action serves all three types, and the object only returns its type and its `Duration`.

The template ships three interactables you can open as examples: `BP_InteractButton` (a button that switches a door), `BP_ItemPickup` (an item lying in the world) and `BP_ReviveBay`.

---

## The component on the player

`BP_InteractionComponent` sits on `BP_FriendslopCharacter`. Every `Trace Interval` it traces from the player's eyes, asks the actor it hits `CanInteract`, and hands the prompt to the HUD. `WBP_Interact`, the ring and text at the centre of the screen, finds the component by itself.

Its fields are in `Settings|Interaction`:

| Field | What it does | Shipped default |
|---|---|---|
| `Trace Distance` | How far the player reaches, in cm | `250` |
| `Trace Interval` | Seconds between two traces | `0.1` |
| `Trace Channel` | The channel the trace uses | `Visibility` |
| `Spam Press Gain` | How much one press adds on a `Spam` object. The ring is full when the total reaches the object's `Duration` | `0.15` |
| `Spam Decay Rate` | How much drains per second on a `Spam` object, in the same unit | `0.4` |
| `Server Distance Tolerance` | The server accepts a use from up to `Trace Distance` times this value | `1.5` |

Two settings on the object itself decide whether it can be used at all:

- It must block the `Trace Channel` (`Visibility` by default). Otherwise it is never looked at and never shows a prompt.
- It must have `Replicates` ticked, even if it has nothing to replicate. The server has to recognise the actor the player sends it.

---

## Who decides

The looking is local. Each player traces, draws the ring and fills it on their own machine. When the ring completes, the component asks the server once. The server checks the distance and `CanInteract` again, and only then calls `Interact` on the object.

So what `Interact` changes must be state the object keeps in a `Replicated` variable. That way every player sees the result, including a player who joins late.

---

## The dispatchers

Bind these to build your own prompt, sounds or tutorial without opening the component.

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnFocusChanged` | The aimed object changed. Gives `Focused Actor`, `Prompt`, `Type` and `Duration`. An empty actor means nothing is aimed | owning player |
| `OnInteractionStarted` | The press began on a target | owning player |
| `OnInteractionCompleted` | The ring completed, before the server has answered | owning player |
| `OnInteractionCanceled` | A `Hold` or `Spam` was abandoned | owning player |

!!! warning
    Never give a reward from `OnInteractionCompleted`. It fires on the player's machine before the server has checked anything, and the server may still refuse. The reward goes in the object's `Interact`.

---

## Buttons, levers and what they switch

A button does not open a door by itself. A second interface, `BPI_Activatable`, is for things a machine switches rather than a player uses. It has two functions: `Toggle` and `SetActivated`.

The link between the two is `BP_ActivationLinkComponent`. It goes on the emitter and holds `Targets`, a list of actors you pick in the level with the eyedropper. When the emitter fires, every target that implements `BPI_Activatable` gets the message. A target that does not implement it simply does nothing.

- `BP_InteractButton` calls `ToggleTargets` from its `Interact`.
- `BP_LeverBase` is dragged by hand, and calls `SetTargetsActivated` when it passes its `Activation Alpha`.
- `BP_DoorBase` implements `BPI_Activatable` and opens or closes.

The link only acts on the server. Called on a player's machine, it does nothing.

Levers and hand-dragged doors are grabbed, not used with `E`. That side is in [How grabbing works](../grab/how_grabbing_works.md).

---

## Where to go next

- [Make any actor interactable](make_an_actor_interactable.md)
- [Link a button or a lever to a door](doors_buttons_and_levers.md)
- [Make your own door, switch or lever](make_your_own_door_or_lever.md)

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
