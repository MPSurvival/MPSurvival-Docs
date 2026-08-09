# Set up doors and the gate button

A door in this template is one Blueprint with three fields on it: which mesh, where the hinge is, where the handle is. Fill those three in and you have a working door that swings, can be locked, can be shot open and comes back the way the player left it after a load.

This page also covers `BP_Interactable_GateButton`, the one shot button that moves a block out of the way. It sits on the same interaction base and does a different job.

- The doors: `Content/TheLastTemplate/Blueprints/Environments/Interactable/Doors/`
- The button: `Content/TheLastTemplate/Blueprints/Environments/Interactable/BP_Interactable_GateButton`

Both derive from the interaction base, so the prompt, the two ranges and the refusal system are the ones described in [How interaction works](how_interaction_works.md). This page only covers what a door adds on top.

---

## What the door base already does

`BP_Interactable_DoorBase` handles all of this for you:

- it builds itself in the construction script from `Door Mesh Asset`, `Hinge Offset` and `Handle Offset`, so the viewport updates as you type
- it moves the prompt onto the handle instead of the actor origin, and flips it to whichever side of the door you are standing on
- it swings open on interact, and swings back on a second interact
- it refuses with the word `Locked` when `Locked` is on
- it can be shot open at the handle when `Can Break Door` is on
- it writes its open state and its locked state into the save

The swing is a 100 degree rotation of the `Door Rotator` component, always the same amount and always the same direction. That number lives in the graph, not on a field, so a door that has to open the other way is a door you turn around in the level.

---

## Make a door from your own mesh

1. In `Doors/Childs/`, duplicate one of the three shipped doors, or right click `BP_Interactable_DoorBase` and create a child Blueprint class. Name it.
2. Open it and set `Door Mesh Asset` to your door panel. This is a `Static Mesh` property, not the mesh on the component: the construction script pushes it into the `Door Mesh` component for you. Leave it empty and the door renders nothing.
3. Set `Hinge Offset`. It is the relative location of `Door Rotator`, the component the whole door turns on. Put it on the hinge edge of your panel.
4. Set `Handle Offset`. It is the relative location of `Door Handle`, which carries the prompt and is the point the break test measures against. Put it on the handle.
5. Type something into `Interact Text`. None of the three shipped doors set one, so a door you place has nothing to say until you do.
6. If your door needs a frame, add a static mesh component for it. Each of the three shipped doors carries one called `Door Frame`.
7. Drag the child into the level.

The two offsets are easier to copy than to guess. The three shipped doors take `SM_Mod_Door_Steel`, `SM_Mod_Int_Door_Panel` and `SM_Mod_Int_Door_Panel_Broken` as their `Door Mesh Asset`, and these are their offsets:

| Door | `Hinge Offset` | `Handle Offset` |
|---|---|---|
| `BP_Interactable_Door_Steel` | `-57.5, 0.75, 0` | `102, 0, 100` |
| `BP_Interactable_Door_Panel` | `-43.4, 0, 1` | `77.8, 0, 100` |
| `BP_Interactable_Door_Panel_Broken` | `-43.4, 0, 1` | `77.8, 0, 100` |

Those three fields are all a door variant sets. The three shipped doors change nothing else on the base.

---

## The fields on a door

| Field | What it does | Default |
|---|---|---|
| `Door Mesh Asset` | The panel mesh, pushed into `Door Mesh` by the construction script | empty |
| `Hinge Offset` | Where `Door Rotator` sits, so where the door turns | `0, 0, 0` |
| `Handle Offset` | Where `Door Handle` sits, so where the prompt shows and where a shot has to land | `0, 0, 0` |
| `Locked` | The door refuses to open | `false` |
| `Can Break Door` | The player can shoot the handle to open the door | `false` |
| `Break Handle Radius` | How close to the handle the shot has to land, in centimetres | `10` |
| `Debug Break Radius` | Debug switch that goes with the radius while you tune it. Turn it off before you ship | `false` |
| `Door State` | Runtime state, read it, do not set it by hand | `Closed` |

Everything else on a door, the prompt text, the icon, the two ranges, comes from the interaction base.

---

## Locked doors

Set `Locked` to true and the door still shows its prompt, but the interaction is refused and the refusal reads `Locked`.

That text is written in the door's `Is Errored` function, not in the `Error Text` field. Changing `Error Text` on a locked door does nothing.

There is no key item in the template. Nothing in the shipped content sets `Locked` back to false except the handle break below, so a locked door is a permanent wall until you write the Blueprint that unlocks it. That is usually enough: a locked door with a window next to it tells the player there is something in that room and they cannot have it yet.

---

## Doors the player can shoot open

Turn on `Can Break Door`. From then on, when point damage lands on the door, three things have to be true:

1. the damage was caused by the player character
2. the hit landed within `Break Handle Radius` of the `Door Handle` position
3. the door is `Closed` or `Closing`

When all three hold, the door sets `Locked` to false and swings open. So this works on a locked door and on an unlocked one.

`Break Handle Radius` ships at `10`, which is 10 cm around the handle. The player has to hit the lock, not the panel. Widen it if your handle mesh is large or if you want the shot to be forgiving.

---

## The four door states

`Door State` uses `E_DoorStates`, whose values are `Closed`, `Opened`, `Closing` and `Opening`.

The interact key only acts on the two resting states: `Closed` opens the door, `Opened` closes it. While the door is `Opening` or `Closing` the key does nothing, so a player cannot flip a door back and forth in the middle of a swing.

The enum is declared with both resting states first and both transitions after, not in cycle order. If you ever read it as a number, it does not step through the animation.

---

## What a door remembers in a save

`BP_Interactable_DoorBase` implements the saveable interface, so a placed door is picked up by the level state without any work from you.

`Capture State` writes three things: whether the door is open (true while `Opened` or `Opening`), whether it is locked, and the actor transform. `Restore State` reads them back, sets the state and places the rotator straight at its end angle instead of replaying the swing. A door the player opened is open when the save loads, and a door they shot open comes back unlocked.

!!! warning
    The save id is the actor's name, returned by `Get Save Id`. Renaming a placed door in the World Outliner after a save has been written breaks the link, and that door loads back at its authored state.

---

## The gate button

`BP_Interactable_GateButton` is a button on a wall that sinks one block into the floor, once, and then refuses to be pressed again. It is used in `L_ShowcaseMap`.

1. Drag `BP_Interactable_GateButton` into the level and put it where the player can reach it.
2. Drag the mesh you want moved into the level as a plain static mesh actor. Set its **Mobility** to **Movable**.
3. Select the button. In its `Gate` field, pick that mesh actor with the eyedropper.
4. Play, walk up, press the interact key.

The button moves the gate's mesh component 420 cm straight down over 1.5 seconds, easing in and out. The distance is in the graph, so size your block around it rather than the other way round.

`Gate` is a **Static Mesh Actor** reference and it is empty on the class. A Blueprint actor will not fit in that slot. The button's `Interact Text` ships as `Open`.

Once it has been used, `Opened` goes true and the button's `Is Errored` starts returning true with the text `Opened`. The prompt switches to its refused look and the player can never press it twice. Nothing resets it inside a play session.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
