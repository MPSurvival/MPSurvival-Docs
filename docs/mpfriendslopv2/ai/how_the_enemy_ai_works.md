# How the enemy AI works, and what it hears

The enemy that ships with the template is the Warden: a robot that patrols, notices you by sight or by sound, and fires a laser at you. It is made of four pieces, and only one of them makes decisions.

- The enemies: `Content/MPFriendslop/Blueprints/AI/`
- The tuning: `Blueprints/DataAssets/Enemy/`
- The noise component: `Blueprints/ActorComponents/BP_NoiseComponent`

This page is the mental model. The pages after it are the recipes.

---

## The four pieces

| Piece | What it owns |
|---|---|
| `BP_EnemyBase` | The body of any enemy. It holds the current state, the suspicion value, the laser aim and the health. Every player sees these values |
| `BP_Warden` | A child of `BP_EnemyBase`. Mesh, animation, laser effect, sounds and camera shakes. It decides nothing, it only shows what the body says |
| `BP_WardenController` | The brain. It runs on the server only: it sees, hears, picks a target, moves and fires |
| `DA_Enemy_Warden` | A `BP_EnemyDataAsset` with the numbers: speeds, ranges, timings, damage. It goes in `Enemy Data` on the enemy |

`BT_Warden` and `BB_Warden` are in `AI/Warden/Behavior/`. The tree is deliberately small: it walks to the point the controller gives it, or waits. All the choices are in `BP_WardenController`, so that is the Blueprint to read.

---

## The states

The current state is an `E_EnemyState`. The controller picks it every frame, on the server.

| State | What the Warden does | What moves it to the next state |
|---|---|---|
| `Patrol` | Walks its `Patrol Points` in order at `Patrol Speed`, pauses `Patrol Wait Seconds` at each one and looks around | Hears a noise: `Investigate`. Sees you for `Detection Seconds`: `Chase` |
| `Investigate` | Walks to the noise | Arrives: `Search` |
| `Chase` | Walks to where it last saw you, at `Chase Speed` | You are within `Attack Range` and visible: `Attack`. Loses sight of you for `Sight Loss Grace Seconds`: `Search` |
| `Attack` | Fires the laser and keeps near `Preferred Combat Distance` | Loses sight of you for `Sight Loss Grace Seconds`: `Search` |
| `Search` | Walks to random points within `Search Radius` of your last known position | `Search Seconds` run out: back to `Patrol` |

A noise heard while it is not patrolling extends the search. All these fields are on `DA_Enemy_Warden`.

---

## What it sees

Sight is the `AIPerception` component of `BP_WardenController`: `Sight Radius` `2200`, `Lose Sight Radius` `2400`, `Peripheral Vision Half Angle Degrees` `65`.

Not everything it sees is a target. A target is a pawn that:

- is controlled by a player,
- returns a `BP_VitalsSystem` from `GetVitalComponent` (`BPI_VitalManagerInterface`),
- is alive, and not down (`GetIsDown` on `BPI_PlayerLifeState`, read from its PlayerState).

Crouching makes a player harder to see. A crouching player is only seen within `Crouched Sight Radius` (`1100`, half the sight radius), and it takes `Crouched Detection Seconds` (`1.5`) instead of `Detection Seconds` (`0.75`) to be spotted. Both are on `DA_Enemy_Warden`. Crouching is read from the pawn's movement component, so it works on any `Character` of your own.

The nearest one wins. A player who dies or goes down is dropped, and the Warden goes to `Search`. The laser damages anything with a vital component that is in the beam, so a friend standing in front of you takes the hit.

The Warden also takes damage, from the guns and melee weapons of the template. At zero health it is removed. There is no death animation. See [How weapons work](../weapons/how_weapons_work.md).

---

## The suspicion arc

While the Warden is building up detection, an arc above its head fills. The arc is sight only: a noise sends the Warden to `Investigate` straight away, without filling it. It is `WBP_EnemyAwareness`, shown by the `AwarenessWidget` component on `BP_EnemyBase`. It only shows between empty and full, and only to a player who can see the Warden. Each player draws it for themselves.

| Field | What it does | Shipped default |
|---|---|---|
| `Arc Color` | Colour of the fill | white |
| `Background Color` | Colour of the empty part | near black, 0.8 opacity |
| `Arc Radius` | Size of the arc | `42` |
| `Arc Thickness` | Thickness of the fill | `3` |
| `Background Thickness` | Thickness of the empty part | `6` |

These are on the widget, in `Settings|Style`. Its position above the head is the `Location` of the `AwarenessWidget` component.

---

## What it hears

The Warden hears through the `Hearing Range` of its `AIPerception`, `2500`. It only listens to noises tagged `Footstep` or `Impact`, and a noise behind a wall is not heard: the server checks the line from the Warden to the noise.

Noises come from `BP_NoiseComponent`. It is on `BP_FriendslopCharacter` and on `BP_LootBase`, and it makes them audible with no graph:

- a player makes a footstep noise every `Footstep Distance` walked on the ground (`Crouch Step Distance` crouched), measured on the server, whatever the animation does;
- a loot item that hits something makes an impact noise, louder the faster it hits.

A noise is heard within `Hearing Range` multiplied by its loudness, and never beyond its max range. The max range of a step follows the player's ground speed: `0` standing still, rising to `Footstep Max Range` at `Footstep Full Range Speed`. With the shipped speeds, a walking step (200 cm/s) is heard at 10 m, a sprinting step (500 cm/s) at 25 m, and a crouched step at about 1 m. The fields are in `Settings|Noise`:

| Field | What it does | Shipped default |
|---|---|---|
| `Footstep Loudness` | Loudness of a step | `1` |
| `Crouch Loudness` | Loudness of a crouched step | `0.05` |
| `Footstep Distance` | Distance walked, in cm, between two steps | `160` |
| `Crouch Step Distance` | The same, crouched | `110` |
| `Footstep Max Range` | Max range of a step at full speed, in cm | `2500` |
| `Footstep Full Range Speed` | Ground speed, in cm/s, at which a step reaches `Footstep Max Range`. Set it to your own character's sprint speed | `500` |
| `Impact Max Range` | Max range of an impact, in cm | `2500` |
| `Impact Reference Speed` | Impact speed, in cm/s, that gives a loudness of 1 | `600` |

To make a noise of your own, an alarm or a slammed door, call `ReportNoise` (`Location`, `Loudness`, `Max Range`, `Tag`) on a `BP_NoiseComponent` on the server, with the tag `Impact`. The Warden ignores any tag other than `Footstep` and `Impact`.

---

## The dispatchers

| Dispatcher | On | When it fires | Fires on |
|---|---|---|---|
| `OnEnemyStateChanged` | `BP_EnemyBase` | The state changed. Read the enemy's state to know the new one. Nothing in the template binds it: it is there for your music, HUD or voice lines | every player |
| `OnNoiseEmitted` | `BP_NoiseComponent` | A noise was reported. Gives `Location`, `Loudness` and `Tag` | server |

---

## Where to go next

- [Place a Warden and its patrol](place_a_warden.md)
- [Doors the Warden can open](doors_the_warden_can_open.md)
- [Make an enemy variant or your own enemy](make_your_own_enemy.md)

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
