# Place a Warden and its patrol

A Warden is one actor, `BP_Warden`, plus the points it walks between. By the end of this page you have a Warden in your level walking its own route, stopping at each point to look around.

- The enemy: `Content/MPFriendslop/Blueprints/AI/Warden/BP_Warden`
- The patrol point: `Content/MPFriendslop/Blueprints/AI/Warden/BP_WardenPatrolPoint`

In `L_Procedural`, you place nothing: the generator spawns the Warden and collects its points, one `BP_WardenPatrolPoint` in every `Normal` and `Passage` room. Follow this page to place a Warden by hand in a map of your own.

---

## Before you start

The Warden walks on the nav mesh. Your level needs a `NavMeshBoundsVolume` that covers the whole route, floor included. Press `P` in the viewport to see the walkable area in green.

---

## The steps

1. Drag `BP_Warden` into the level where it should start.
2. Drag in a `BP_WardenPatrolPoint` for each stop of the route. Put each one on the floor, inside the green area.
3. Rotate each point so its arrow faces what the Warden should watch. While it waits at a point, the Warden looks from side to side, about 45 degrees each way around that arrow.
4. Select the Warden. Under `Settings|Patrol`, add one entry per point to `Patrol Points` and pick each point with the eyedropper, **in walking order**.
5. Check that `Enemy Data` under `Settings|Enemy` is set. `BP_Warden` ships with `DA_Enemy_Warden`.

The Warden walks the points in order and goes back to the first after the last. Any `TargetPoint` works in the array: `BP_WardenPatrolPoint` is a `TargetPoint` with no logic of its own.

With `Patrol Points` empty, the Warden stands where you placed it until it sees or hears something.

---

## The fields

| Field | Where | What it does | Default |
|---|---|---|---|
| `Patrol Points` | placed `BP_Warden`, `Settings\|Patrol` | The route, in walking order | empty |
| `Enemy Data` | `BP_Warden`, `Settings\|Enemy` | The Data Asset with all the tuning | `DA_Enemy_Warden` |
| `Patrol Speed` | `DA_Enemy_Warden` | Walking speed on the route, in cm/s | `90` |
| `Patrol Wait Seconds` | `DA_Enemy_Warden` | Pause at each point | `2.5` |
| `Arrival Radius` | `BP_WardenController` Class Defaults, `Settings\|Patrol` | Distance, in cm and measured flat, at which a point counts as reached | `130` |

`DA_Enemy_Warden` is shared by every Warden. To give one Warden a slower or longer patrol, duplicate it, change the copy and set it in `Enemy Data` on that placed Warden. The other fields of the Data Asset are on the next page.

---

Next: [Doors the Warden can open](doors_the_warden_can_open.md), for the doors on its patrol.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
