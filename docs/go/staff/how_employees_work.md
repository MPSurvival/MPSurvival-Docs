# How employees work

Employees do not look for work. Work is published to a queue, and a free employee takes the highest priority job they know how to do. That is what lets you add a new job without touching the AI.

---

## Who does what

`RoleTaskTable` on `BP_StoreManager` maps a role to the task types it handles. It is in the Details panel, in `Settings|Tasks`:

| Role | Task types |
|---|---|
| `Cashier` | `ManCheckout` |
| `Stocker` | `StockShelves` |

`Roles` on a candidate is an **array, and its order is the priority**. Chloé is `[Cashier, Stocker]` and Dmitri is `[Stocker, Cashier]`: same two skills, different first choice.

Work is published by the things that need it:

| Producer | Task | When |
|---|---|---|
| `BP_CheckoutBase` | `ManCheckout` | At `BeginPlay`, and whenever the till has no cashier |
| `BP_DeliveryBoxBase` | `StockShelves` | When it holds units and the player is not carrying it |

A box you are holding never publishes, so you always win a tug of war over a box.

---

## The cashier

Walks to the till's `CashierSpot` behind the counter, claims it, and the scanner attaches to their hand — if you were holding it, it is taken from you, and you cannot take it back while the till is manned. They scan one item every `CashierScanSeconds` (one second) and take the payment when the basket is done, with no drawer and no counting. An employee never miscounts, and that is what you pay them for.

Going off shift releases the till and the scanner. It does not close the till.

---

## The stocker

Takes the task published by a box, works out where the contents go — shelves first, then a storage rack — picks the box up, and unloads one unit at a time into the shelf. When the shelf fills or the box empties, they retarget to another shelf, then a rack, then put the box down.

If nothing at all will take the contents, the box is left where it is.

---

## Shifts and wages

The roster lives on `BP_StaffManager`, on the game state. Each line is a candidate, a shift and the actor in the world. The line survives the night, when no employee exists in the level at all.

A shift is:

| Field | What it is |
|---|---|
| `DayMask` | Seven bits, one per weekday. `127` is every day |
| `StartHour` | When they clock on |
| `EndHour` | When they clock off |

People are spawned and despawned at `BP_StaffEntrance` as the hours pass, and wages are charged at the end of each day.

**Hiring and schedule changes take effect the next day.** The Staff app edits the shift that starts tomorrow, so someone hired at two in the afternoon starts tomorrow morning.

---

## Adding a job

1. Add an entry to `E_EmployeeRole`.
2. Add an entry to `E_TaskType`.
3. Add a row to `RoleTaskTable` joining them.
4. Add a `Work<Task>` function on `BP_EmployeeBrain`, and a branch on its `PerformTask` switch.
5. Call `PublishTask` with that type from wherever the work appears.

Steps 1 to 3 are done in the Details panel. Step 5 is the one people forget: a task type nobody publishes gives you an employee qualified for a job that never comes up.

Your function returns an outcome when it is done:

| Outcome | What happens |
|---|---|
| `Completed` | The task leaves the queue |
| `Continuing` | The employee keeps it and keeps working |
| `Refused` | The task goes back in the queue for someone else |

`Continuing` is what makes a job last longer than one trip: manning a till returns it for as long as the employee stands there, so a cashier does not wander off between customers.

!!! warning
    Write `RoleTaskTable` in **both** places: the `BP_StoreManager` class defaults, and the component template on `BP_StoreGameState`. The template is what runs.

---

## The carry animation

A box held by an employee is attached to a socket on `hand_r`, with the upper body blended over the walk so the legs keep going normally. The left hand is placed on the box by an IK target that comes from the box itself, so a small box and a large box are both held correctly.

The carry pose and the hand offset are settings on the employee brain; the blend speed is `CarryBlendSpeed` on `ABP_Humanoid`.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
