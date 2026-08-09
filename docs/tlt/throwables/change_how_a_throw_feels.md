# Change how a throw feels

How far a throwable flies, how hard it arcs, whether it bounces or breaks, and what the camera does while you line the shot up: all of it is Details panel fields on one Data Asset. You change how a throw feels by opening that asset and typing numbers. No graph.

The six shipped throwables live in `Content/TheLastTemplate/Blueprints/DataAssets/Throwables/Childs/`. They are named `DA_Throwble_*`, missing an "a", in the shipped project. Open one and everything on this page is in front of you.

If you want to know what the four assets behind a throwable are and what happens on impact, read [How throwables work](how_throwables_work.md) first. This page only covers the feel.

---

## What actually differs between the six

Almost nothing. Two fields carry the whole difference:

| Data Asset | `Launch Speed` | `Destroy on Impact` | `Bounce Sound` and `Hit Sound` |
|---|---|---|---|
| `DA_Throwble_Bomb` | `1700` | ticked | `SC_BreakGlass` |
| `DA_Throwble_Bottle` | `1700` | ticked | `SC_BreakGlass` |
| `DA_Throwble_Brick` | `1200` | unticked | `SC_BrickHit` |
| `DA_Throwble_Can` | `1400` | unticked | `SC_MetalHit` |
| `DA_Throwble_Molotov` | `1700` | ticked | `SC_BreakGlass` |
| `DA_Throwble_NailBomb` | `1700` | ticked | `SC_MetalHit` |

Every other field on this page holds the same value on all six. So if a change you make feels wrong on one throwable, it will feel wrong on the other five too until you edit them as well.

---

## Flight

Three fields decide the shape of the arc. They are pushed onto the projectile movement of the thrown actor the moment it spawns.

| Field | What it does | Shipped |
|---|---|---|
| `Launch Speed` | Speed the throwable leaves your hand at, in centimetres per second | `1200` to `1700` |
| `Pitch Offset` | Degrees added above your aim line, so the throw lobs instead of going flat | `8` |
| `Gravity Scale` | Multiplier on gravity while it flies. Above `1` it drops faster | `1.25` |

`Launch Speed` is the one to reach for first. It moves distance and flight time together and you feel it immediately. Only touch `Pitch Offset` when the throw goes where you want but looks too flat or too high, and only touch `Gravity Scale` when you want the object to feel heavier or lighter than the speed alone suggests.

The trajectory arc is drawn from the same three numbers, so it stays honest whatever you set.

---

## Landing

| Field | What it does | Shipped |
|---|---|---|
| `Bounciness` | How much speed survives a bounce. `0` sticks, `1` keeps everything | `0.15` |
| `Friction` | How fast it loses speed sliding along a surface | `0.6` |
| `Collision Radius` | Radius in centimetres of the sphere that detects the impact | `7` |
| `Destroy on Impact` | Whether the first bounce ends the throw | ticked on four of six |

`Destroy on Impact` is the one that changes the object into a different kind of thing. Ticked, the throwable fires its payloads on the first bounce and is removed: that is the bomb, the nail bomb, the molotov and the bottle. Unticked, it keeps bouncing and rolling and the object is left lying in your level: that is the brick and the can.

`Collision Radius` at `7` is small on purpose. A large radius makes a throwable detonate early on door frames and railings it should have flown past.

---

## The aiming camera

Raise a throw and the camera stops using the resting values and uses these instead. They sit on the throwable's Data Asset, not on the character, so each throwable could frame its own aim. All six ship with the same numbers.

| Field | What it does | Shipped |
|---|---|---|
| `Spring Arm Right` | Where the camera sits while aiming. Y across the shoulder, Z up and down | `(0, -60, 0)` |
| `Spring Arm Right Crouching` | The same thing, used while crouched | `(0, -60, -35)` |
| `Zoom Value` | Spring arm length while aiming. Smaller means closer to the shoulder | `110` |
| `Camera FOV` | Field of view while aiming | `90` |

The throw camera pushes to `-60` on Y and back out to `110` while a gun pulls to the other shoulder and in close. That is deliberate: when you throw, you need to see the landing spot more than you need to see the character. If you copy camera numbers from a weapon Data Asset onto a throwable, expect the arc to disappear behind the shoulder.

---

## While you aim

| Field | What it does | Shipped |
|---|---|---|
| `Aim Movement Speed` | Movement speed multiplier while aiming a throw | `0.55` |
| `Enable Aim Immersive Camera` | Turns the slow aim sway on | ticked |
| `Aim Camera Intensity` | How strong that sway is | `1.5` |

`0.55` is slower than the guns, which use `0.65`. Lining up a throw is meant to commit you more than raising a pistol does. Untick `Enable Aim Immersive Camera` if you want a dead still camera, and `Aim Camera Intensity` stops mattering.

---

## The trajectory arc

`Show Trajectory` is a single tick box on the Data Asset, on for all six. Untick it and that throwable is aimed blind, with no arc drawn. It is a good way to make one item harder to use than the rest.

The arc itself is drawn by `Content/TheLastTemplate/Blueprints/Throwables/BP_TrajectoryVisualizer`. Open it and its Details panel gives you the shape:

| Field | What it does | Shipped |
|---|---|---|
| `Max Segments` | How many pieces the arc can use, so how far ahead it draws | `80` |
| `Segment Length` | Length in centimetres of one step of the simulated path | `20` |
| `Start Width` | Thickness of the tube at the hand end | `0.15` |
| `End Width` | Thickness at the far end | `0.15` |

Eighty segments of twenty centimetres is about sixteen metres of drawn path. If your throwable flies further than that, the arc stops before the landing point. Raise `Max Segments` rather than `Segment Length`, because a longer step makes the curve visibly faceted.

Set `End Width` lower than `Start Width` for an arc that tapers away, which reads well when you want the far end to feel like a guess.

---

## Restyling the arc

The tube mesh and its material are assigned from inside `BP_TrajectoryVisualizer`, so the way to restyle the arc without opening a graph is to edit the two assets in place:

- `Content/TheLastTemplate/Meshes/Misc/SM_ArcTube` is the tube. Replace its geometry and every segment follows.
- `Content/TheLastTemplate/Materials/Effects/MI_ThrowArc` is the material. It is a child of `M_ThrowArc` and exposes `Intensity`, shipped `2.5`, and `Opacity`, shipped `1`.

Lower `Intensity` for an arc that does not glow through dark rooms. Lower `Opacity` for one you can see the level through.

---

## Sounds

| Field | When it plays | Shipped |
|---|---|---|
| `Throw Sound` | As the throwable leaves your hand | empty on all six |
| `Bounce Sound` | When the flying throwable bounces off something | per throwable |
| `Hit Sound` | When the object lying in the level is knocked around | per throwable |
| `Hit Cosmetic Sound Threshold` | How hard that knock has to be before `Hit Sound` plays | `20000` |

`Throw Sound` is empty on every shipped throwable, so nothing plays on the swing today. The field works, it is simply not filled in. Drop a cue in it if you want one.

`Bounce Sound` and `Hit Sound` are two different moments and the shipped assets set them to the same cue. `Bounce Sound` belongs to the object while it is still flying under its own launch. `Hit Sound` belongs to the object afterwards, once it is a physics prop being kicked around, and that is where `Hit Cosmetic Sound Threshold` comes in: the impact has to be stronger than `20000` before the sound plays. That is what keeps a can from clicking on every tiny settle. Lower it and light taps become audible, raise it and only real impacts speak.

---

## Animation and the hand

| Field | What it does | Shipped |
|---|---|---|
| `Throw Montage` | The animation played on the character | `AM_Character_Throw` |
| `Throwable Socket Name` | Socket the object is held on and thrown from | `ThrowableSocket` |

All six throwables share `AM_Character_Throw`. The object does not leave the hand when the montage ends, it leaves on an anim notify inside the montage called `Release`. If a throw looks late or early, move that notify, do not change `Launch Speed` to compensate.

`ThrowableSocket` is a socket on the skeleton `Content/TheLastTemplate/Meshes/Characters/S_HumanoidCharacter`. Type a socket name that does not exist there and the object is held at the character's feet.

Give a throwable its own montage by pointing `Throw Montage` at your own asset. Copy the `Release` notify onto it, otherwise the object is never let go.

---

## Worked example: a heavy lob against a light long throw

Duplicate `DA_Throwble_Brick` twice and give each copy one of these sets.

A heavy short lob, for something you drop over a wall rather than throw across a street:

| Field | Value |
|---|---|
| `Launch Speed` | `900` |
| `Pitch Offset` | `20` |
| `Gravity Scale` | `2.0` |
| `Bounciness` | `0.05` |
| `Friction` | `0.9` |
| `Destroy on Impact` | unticked |

A light long throw, for something you want to skitter down a corridor and pull attention far away:

| Field | Value |
|---|---|
| `Launch Speed` | `2200` |
| `Pitch Offset` | `4` |
| `Gravity Scale` | `0.9` |
| `Bounciness` | `0.5` |
| `Friction` | `0.25` |
| `Destroy on Impact` | unticked |

Also raise `Max Segments` on `BP_TrajectoryVisualizer` before you test the second one, or its arc will be cut off well short of where it lands.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
