# The six rules the template is built on

Six ownership decisions were taken before the first line was written, and every system in the project obeys them. If you extend the template, they are the part worth keeping.

They exist because the previous template in the studio's catalogue was refactored three times mid-project after several systems ended up writing the same piece of state.

---

## D1: Money has one writer

`BP_StoreManager.AddTransaction(Amount, Reason, ProductId)` is the only place in the project where the balance changes. Everything else calls it.

Every call lands in a journal with a reason attached, which is what makes the daily report **derived** rather than accumulated. A second writer, and the report starts lying.

---

## D2: Time has one owner

`BP_StoreManager` holds the day and the hour. Nobody else advances time, and no gameplay logic runs on a timer. The hour is broadcast with `OnHourChanged` and the day with `OnDayEnded`, and everything subscribes.

That is why freezing the clock at closing time freezes the sun, the deliveries and the spawner without a line being written in any of them.

A related trap: **world timers do not run while the game is paused.** Anything a paused screen depends on has to run from a widget `Tick` or be called directly.

---

## D3: One storage component, used by three things

The shelf, the storage rack and the delivery box all use the same storage machinery. Same placement, same stacking, same rules.

Three implementations would be three behaviours that diverge at the first bug fix.

---

## D4: A customer's target has one writer

`BP_CustomerBrain.UpdateTarget()` is the only thing that writes where a customer is going and what they are looking at. It is called from a service on the root of the behaviour tree, so it runs in every state.

**No task touches the target or the focus.** A task writes a value into the brain and the brain decides.

The employee brain follows the same shape.

---

## D5: Employees pull from a task queue

Work is published to a queue on `BP_StoreManager`. A free employee takes the highest priority task they are qualified for. They do not go looking for work.

That is what lets you add a job without touching any existing AI, and it is what will make the multiplayer version possible without rewriting the logic.

---

## D6: The view has one writer

`BP_ViewMotionComponent` is the only thing that writes the view root's position and the camera's rotation. The camera has `bUsePawnControlRotation` **off**, and the component recomposes the control rotation's pitch plus its own procedural offsets every tick.

Anything that wants to move the view goes through an input on that component. Nothing calls `SetRelativeRotation` on the camera from the side.

That is why leaning into a screen still has head bob and breathing living on top of it, and why it does not fight the camera manager.

---

## Two habits that go with them

**Drive the native system rather than writing the value yourself.** Before writing gameplay logic, find which subsystem already owns the quantity: the character movement component for velocity and rotation, the AI controller for focus, the nav mesh for pathing, the attachment graph for keeping a widget beside an object. Then drive it with its own flags.

A per-frame writer is justified only after you have established that no native owner exists, and it should be said out loud when it happens.

**A persistent native state gets one owner that re-asserts it every tick.** Not several places that clean up after themselves. The sun's rotation, a customer's focus, the walk speed. Decide the owner on the day the system is born, not on the day the bug appears.

---

## What follows from all of this for you

If you add a system:

- Route its money through `AddTransaction`.
- Subscribe to `OnHourChanged` or `OnDayEnded` rather than starting a timer.
- If it involves storing goods, reuse the storage component.
- If it involves an AI decision, put it in the brain and leave the tree thin.
- If it involves the camera, add an input to the view component.
- Decide who owns each piece of state before writing the first node.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
