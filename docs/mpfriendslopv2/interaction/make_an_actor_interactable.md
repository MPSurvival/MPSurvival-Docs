# Make any actor interactable

By the end of this page, any actor of yours shows a prompt when a player looks at it, and does something when they press `E`. You implement one interface on the actor. The player character is not touched.

If you have not read [How interaction works](how_interaction_works.md), start there.

- The interface: `Content/MPFriendslop/Blueprints/Interfaces/BPI_Interactable`
- The example to copy from: `Blueprints/Environments/Fixtures/BP_InteractButton`

---

## The recipe

1. Open your actor Blueprint and click **Class Settings**.
2. Under `Implemented Interfaces`, click **Add** and pick `BPI_Interactable`.
3. In **My Blueprint**, under **Interfaces**, open `GetInteractionData`. Return the three values:
    - `Prompt`: the line shown under the ring, for example "Open Crate".
    - `Type`: `Simple`, `Hold` or `Spam`.
    - `Duration`: for `Hold`, the hold time in seconds. For `Spam`, how full the ring must get (see below). Ignored for `Simple`.
4. Open `CanInteract` and return true when the object may be used right now. Return false when it is locked, already used, or busy.
5. Add the `Interact` event. This is where the object does its work. It runs on the server.
6. In `Interact`, change a variable set to `Replicated` or `RepNotify`, and let that variable drive what the players see. [Rules for your own additions](../start/how_multiplayer_works.md#rules-for-your-own-additions) says why.
7. Compile and save, place the actor in the level, and look at it.

`StartFocus` and `EndFocus` are optional. They fire on the looking player's machine only, so use them for a cosmetic highlight and nothing else.

A good habit is to expose the prompt, type and duration as variables on the actor, in a `Settings|<Sub-category>` of your own, and return them from `GetInteractionData`. That is what `BP_InteractButton` does with its `Prompt`, `Type` and `Duration` fields, so each placed button can say something different.

---

## The actor settings that break it silently

| Setting | Where | If it is wrong |
|---|---|---|
| `Replicates` | Class Defaults, **Replication** | The server cannot find the actor the player used, and nothing happens |
| Collision on the `Visibility` channel | the mesh's `Collision Presets` | Set to anything but **Block**, the player's trace goes through: no prompt, no use |

!!! warning
    Tick `Replicates` even when your actor has nothing else to replicate. The player's request names the actor it used, and the server can only read that name for a replicated actor. Without it, the prompt and the ring work on the player's screen and the use is simply lost, with no error.

---

## Hold or mash

You do not add a trigger or a timer for either. The object returns its `Type`, and the one `IA_Interact` action handles all three.

| Type | What the player does | What you set |
|---|---|---|
| `Simple` | One press | nothing else |
| `Hold` | Holds `E` until the ring fills. Looking away or stepping out of range cancels it | `Duration`, in seconds |
| `Spam` | Presses `E` again and again. Each press fills the ring a little, and it drains between presses | `Duration`, the amount to reach: each press adds `Spam Press Gain` (0.15 by default), so `1.2` takes eight quick presses |

How much each press adds and how fast the ring drains is set on the player, not on the object: `Spam Press Gain` and `Spam Decay Rate` on `BP_InteractionComponent`, in `Settings|Interaction`. They apply to every `Spam` object the player uses.

---

## Add the outline

Interactive objects in the template carry a thin dark outline, so players can tell them from scenery. It is a permanent marker, not a highlight that appears when you look at the object. Your actor opts in with one component.

1. Add a `BP_OutlineComponent` to your actor.
2. In `Mesh Names`, list the names of the mesh components to outline, exactly as they appear in the **Components** panel. The default is `Mesh`.
3. Make sure your level has a post process volume with `Infinite Extent (Unbound)` ticked and `MI_InteractableOutline` in its blendables. `L_Procedural` has one, labelled `PostProcess_InteractableOutline`, in the outliner folder `Lighting`. Copy it into your own level.

The look of the line (thickness, colour, opacity, fade with distance) is on `MI_InteractableOutline`, in `Materials/PostProcess/`, and applies to every outlined object at once.

Two things to know about the outline:

- It is written when the game starts. To see it in the editor viewport too, tick `Render CustomDepth Pass` and set `CustomDepth Stencil Value` to `1` on the mesh in your Class Defaults.
- At most 255 actors can be outlined at the same time.

---

To make your object switch a door or another actor instead of doing the work itself, see [Link a button or a lever to a door](doors_buttons_and_levers.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
