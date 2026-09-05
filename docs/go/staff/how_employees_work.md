# How employees work

Employees do not look for work. Work is published to a queue, and a free employee takes the highest priority job they know how to do.

That one decision is why you can add a new job to this template without touching the AI.

---

## The task queue

`BP_StoreManager` owns the queue. A task is an `S_StoreTask`:

| Field | What it is |
|---|---|
| `Type` | An `E_TaskType` |
| `Target` | The actor the job is about |
| `Priority` | Higher wins |
| `AssignedTo` | Who took it, or nothing |

Two functions move tasks around: `PublishTask` puts one in, `PullTaskForRoles` takes the best one an employee can do.

**What a role can do is data, not a switch.** `RoleTaskTable` on `BP_StoreManager` is a list of `S_RoleTasks` rows, each mapping a role to the task types it handles:

| Role | Task types |
|---|---|
| `Cashier` | `ManCheckout` |
| `Stocker` | `StockShelves` |

It is Instance Editable, in `Settings|Tasks`, so you edit it in the Details panel.

When a task finishes, the employee returns an `E_TaskOutcome`:

| Outcome | What happens |
|---|---|
| `Completed` | The task leaves the queue |
| `Continuing` | The employee keeps it and keeps working |
| `Refused` | The task goes back in the queue for someone else |

`Continuing` is what lets a job last longer than one trip. Manning a till returns `Continuing` for as long as the employee is standing at it, which is why a cashier does not wander off between customers.

---

## An employee has skills, not a job

`Roles` on a candidate is an **array**, and the order is the priority. Chloé is `[Cashier, Stocker]` and Dmitri is `[Stocker, Cashier]`: they can do exactly the same two things, and they reach for different ones first.

---

## Who publishes work

| Producer | Task | When |
|---|---|---|
| `BP_CheckoutBase` | `ManCheckout` | At `BeginPlay`, and again whenever the till has no cashier |
| `BP_DeliveryBoxBase` | `StockShelves` | When it holds units and the player is not carrying it |

A till that already has a cashier stops asking. Without that guard it re-published every game hour and a second employee kept coming to take the post off the first.

A box the player is holding never publishes. You always win a tug of war over a box.

---

## The cashier

1. Take the `ManCheckout` task and walk to the till's `CashierSpot`, which is behind the counter, not in front of it.
2. Claim the till. `TryClaimCashier` refuses if someone else already has it and accepts the same employee re-asserting.
3. The scanner attaches to their hand. If you were holding it, it is taken from you, and you cannot pick it up again while the till is manned.
4. Scan one item every `CashierScanSeconds`, one second by default.
5. Take the payment when the last item is scanned. No drawer, no counting, no card terminal. An employee is faster than you and never miscounts, and that is what you are paying for.

"This till is manned" has one writer, the till itself, in a `Cashier` variable. `IsManned()` reads it, and it is written the day the state was created rather than bolted on later.

An employee going off shift releases the till and the scanner in `EndPlay`. Releasing the post does **not** close the till, which was a bug worth remembering: closing it stopped every customer from entering, in front of a player who had done nothing wrong.

---

## The stocker

1. Take the `StockShelves` task published by a box.
2. Work out where the contents should go. Shelves first, the first one whose `HasRoomFor` says yes; then a storage rack; and if nothing at all will take it, the task completes and the box is left where it is.
3. Pick the box up. It is **attached to a hand socket**, not held by the physics grab the player uses, with its physics and collision switched off.
4. Walk to the shelf and unload one unit per beat, using the same `UnloadOneInto` your left mouse button calls.
5. When the shelf fills or the box empties, retarget: another shelf, then a rack, then put the box down.

"A shelf that is empty or already holds this product" needed no extra code, because that is what `HasRoomFor` already means: the category is accepted, the row is empty or matching, and there is headroom.

A rack that has just refused is not tried again, so there is no loop.

---

## Carrying, animated

An employee carrying a box is not the same rig as the player. The box is attached to a socket on `hand_r`, and the animation is a `Layered blend per bone` filtered on `spine_01`, so the legs keep walking normally while the upper body holds the box.

The **left** hand is placed on the box by a single `Two Bone IK`, and its target comes from the box itself rather than a tuning value, so a small box and a large box are both held correctly.

Two knobs on the employee brain, in `Settings`, adjust the fit: the carry pose and the hand offset. The blend speed between carrying and not carrying is `CarryBlendSpeed` on `ABP_Humanoid`.

---

## Shifts and wages

`BP_StaffManager` is a component on the game state, next to the other managers. It owns the roster and nothing else owns any part of it.

A roster line is an `S_HiredEmployee`: the candidate, their shift, and the spawned actor. The line survives the night, when no employee exists in the world at all.

A shift is an `S_Shift`:

| Field | What it is |
|---|---|
| `DayMask` | Seven bits, one per weekday. `127` is every day |
| `StartHour` | When they clock on |
| `EndHour` | When they clock off |

`EvaluateShifts` runs on every `OnHourChanged` and spawns or despawns people at `BP_StaffEntrance`. `PayWages` runs on `OnDayEnded` and charges the sum of the `DailyWage` of everyone who worked, with reason `Salary`.

**Hiring and schedule changes take effect the next day.** A roster line carries a `NextShift` alongside its live `Shift`, the Staff app edits `NextShift`, and `ApplyNextShifts` copies it over at the end of the day. Someone hired at two in the afternoon starts tomorrow morning, which is how hiring works.

---

## Adding a job

The whole point of the design. A new job is:

1. A new entry in `E_EmployeeRole`.
2. A new entry in `E_TaskType`.
3. A row in `RoleTaskTable` joining them.
4. A `Work<Task>` function on `BP_EmployeeBrain`, and a branch on its `PerformTask` switch.
5. Something, somewhere, that calls `PublishTask` with that type.

Steps 1 to 3 are done in the Details panel. Step 5 is the one people forget: a task type nobody publishes gives you an employee who is qualified for a job that never appears.

Write `RoleTaskTable` in **both** places, the `BP_StoreManager` class defaults and the component template on `BP_StoreGameState`. The template shadows the class defaults, and the template is what runs.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
