# Make an enemy variant or your own enemy

There are three levels of change. A **variant** is a new Data Asset: a faster, deadlier or more patient Warden, with no graph opened. A **new body** is a child of `BP_Warden` with your own mesh, running the same brain. A **new attack** needs a controller of your own.

If you have not read [How the enemy AI works](how_the_enemy_ai_works.md), start there.

- Warden Blueprints: `Content/MPFriendslop/Blueprints/AI/Warden/`
- Enemy base class: `Blueprints/AI/Base/BP_EnemyBase`
- Enemy Data Assets: `Blueprints/DataAssets/Enemy/Childs/`

---

## A variant, from a Data Asset

1. In `Blueprints/DataAssets/Enemy/Childs/`, duplicate `DA_Enemy_Warden`. Name it, for example `DA_Enemy_WardenHunter`.
2. Change the numbers you want.
3. Right click `BP_Warden` and create a child Blueprint, for example `BP_WardenHunter`.
4. In its **Class Defaults**, set `Enemy Data` to your Data Asset.

For a single Warden that differs from the others, you can skip steps 3 and 4: select the placed Warden and set `Enemy Data` on it.

| Field | What it does | `DA_Enemy_Warden` |
|---|---|---|
| `Detection Seconds` | Time the enemy must keep a player in sight before it chases. The arc above its head fills over this time | `0.75` |
| `Crouched Sight Radius` | Distance, in cm, beyond which a crouching player is not seen at all | `1100` |
| `Crouched Detection Seconds` | The same as `Detection Seconds`, for a crouching player. Keep it above `0` | `1.5` |
| `Sight Loss Grace Seconds` | Time out of sight before it gives up and starts searching | `1` |
| `Search Seconds` | How long it searches before going back to its patrol. A new noise extends it | `10` |
| `Search Radius` | Radius, in cm, around the last known position where it picks search points | `500` |
| `Patrol Speed` | Walk speed while patrolling and investigating a noise, in cm/s | `90` |
| `Chase Speed` | Walk speed while chasing and attacking, in cm/s | `90` |
| `Patrol Wait Seconds` | Pause at each patrol point | `2.5` |
| `Attack Range` | Distance at which it starts firing. It is also the length of the laser, in cm | `1100` |
| `Preferred Combat Distance` | Distance it tries to keep while firing, in cm. Capped at 80 % of `Attack Range` | `600` |
| `Combat Distance Tolerance` | How far it lets the distance drift before it moves, in cm | `75` |
| `Laser Damage` | Health removed at each damage tick | `10` |
| `Damage Interval` | Seconds of unbroken contact with the beam per damage tick. The count starts again if the beam leaves the target | `0.3` |
| `Aim Interp Speed` | How fast the aim follows the target. Lower is easier to dodge | `2` |
| `Aim Sway Degrees` | Wobble added to the aim | `1` |

The Warden only walks, which is why both speeds are `90`. A higher `Chase Speed` gives you a hunter that closes in. The walk animation is authored for the walk speed, so check how the feet look at your new speed.

---

## What it sees and hears

Sight and hearing ranges are not on the Data Asset. They are on the `AIPerception` component of `BP_WardenController`.

1. Right click `BP_WardenController` and create a child Blueprint, for example `BP_WardenHunterController`.
2. Select its `AIPerception` component and open `Senses Config`.
3. Change the values below.
4. In your enemy's **Class Defaults**, set `AI Controller Class` to your new controller.

| Field | What it does | `BP_WardenController` |
|---|---|---|
| `Sight Radius` | How far it sees, in cm | `2200` |
| `Lose Sight Radius` | How far a seen player must go to be lost, in cm | `2400` |
| `Peripheral Vision Half Angle Degrees` | Half of its field of view | `65` |
| `Hearing Range` | How far it hears, in cm | `2500` |

The controller also has `Arrival Radius` (`130`, how close counts as "arrived" at a patrol point or a noise) and `Laser Activation Delay` (`0.5`, seconds of line of sight before each burst), in its **Class Defaults**.

---

## A new body

Keep the brain, change the look. Start from `BP_Warden`, not from `BP_EnemyBase`: `BP_Warden` is the one that draws the laser, plays its loop and its shakes.

1. Create a child Blueprint of `BP_Warden`.
2. Select `Mesh`. Set `Skeletal Mesh Asset` to your mesh and `Anim Class` to your animation Blueprint.
3. Check that your mesh has a socket named `LaserMuzzle`. The beam starts there and fires along the socket's X axis. On `SKM_Robot_Warden` it sits on `lowerarm_r`.
4. Keep `AI Controller Class` on `BP_WardenController`, or on your child of it, and `Auto Possess AI` on **Placed in World or Spawned**. Both are already set when you start from `BP_Warden`.

`SKM_Robot_Warden` uses `SKEL_Mannequin`. A mesh on the same skeleton can keep `ABP_Warden` as it is. For another skeleton, build an animation Blueprint that reads these from the pawn:

- the velocity, for walking
- `Aim Rotation`, where the laser points, for an aim offset
- `Laser Active`, true while it fires, to blend the aim pose in

These values are replicated, so an animation Blueprint that only reads them looks the same on every player's screen.

The laser look and sound are fields on the child, under `Settings|Laser` and `Settings|Audio`:

| Field | What it does | `BP_Warden` |
|---|---|---|
| `Laser Transition Duration` | Time for the beam to extend and retract. Also the fade of the laser loop | `0.25` |
| `Laser Endpoint Interp Speed` | Smoothing of the beam's end point | `25` |
| `Laser Impact Shake Class` | Camera shake near where the beam lands | `CS_WardenLaserImpact` |
| `Footstep Sound` | Played at each step | `CUE_Warden_Footstep` |

The effect itself is `NS_Warden_Laser` on the `LaserVFX` component, and the loop is `CUE_Warden_Laser_Loop` on `LaserAudio`.

!!! warning
    If your child overrides **Event BeginPlay**, keep the **Parent: BeginPlay** call. That call is what connects the suspicion arc to the enemy. Without it the arc never shows, and nothing tells you why.

---

## Its footsteps, as an example of a hookup

`BP_Warden` shows how to react to something on the body without touching the brain. Its `BP_FootstepComponent` fires `OnFootstep` at each foot contact. The Warden answers by moving its `FootstepShake` component to the foot, starting the `CS_WardenFootstep` camera shake and playing `Footstep Sound` there.

Steps come from `BP_FootstepNotify` on the walk animations. Your own walk cycle needs those notifies for the stomp to play. See [Sounds, footsteps and surfaces](../world/sounds_and_footsteps.md).

---

## Health and death

The enemy's health is the `BP_VitalsSystem` component it inherits from `BP_EnemyBase`. It takes Unreal's own damage, so the shotgun and melee weapons already hurt it. Its `Vitals` row uses `DA_Health_Vital_Enemy`, in `Blueprints/DataAssets/Vitals/Childs/`: `200` health, separate from the players' `DA_Health_Vital`.

To give your enemy its own health:

1. Duplicate `DA_Health_Vital_Enemy` and set `Vital Max Amount` on the copy. The original is shared by every enemy.
2. In your enemy, select `BP_VitalsSystem`. In the `Vitals` row, set the `Vital Asset` to your copy and `Current Amount` to the same value.

At zero health the component calls `OnDeath` from `BPI_VitalManagerInterface` on the enemy. `BP_EnemyBase` answers by destroying the enemy, on the server. There is no death animation and no ragdoll. To add one, or to drop loot, override `OnDeath` in your child Blueprint and do it there. If you do not call the parent, the enemy is no longer destroyed, and removing it becomes your job.

!!! warning
    A Warden already placed in a level can keep its own component values. After you change `BP_VitalsSystem` on the class, select each placed enemy and check its `Vitals` row too. A placed enemy with an empty `Vitals` takes its hits on zero health. There is no error.

---

## A different attack

The template ships one attack: the laser. It is written into `BP_WardenController`, and the values it replicates (`Laser Active`, the beam's start and end) live on `BP_EnemyBase`. There is no switch that turns it into a melee attack or a projectile.

For another attack, make a child of `BP_EnemyBase` with your own AI controller, and reuse the same pattern. The controller decides on the server. It writes the enemy's state into replicated variables on the pawn. The pawn draws the result on every screen. `Enemy Data`, `Patrol Points`, the health component and the suspicion arc all come with `BP_EnemyBase`.

To follow the state from outside, for music or a HUD warning, bind `OnEnemyStateChanged` on the enemy and read its `Enemy State` (`E_EnemyState`).

| Dispatcher | When | Fires on |
|---|---|---|
| `OnEnemyStateChanged` | The enemy moves to Patrol, Investigate, Chase, Attack or Search | every player |

---

To put your enemy in a level with a route to walk, see [Place a Warden and its patrol](place_a_warden.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
