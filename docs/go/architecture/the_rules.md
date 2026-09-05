# Extending the template

Six rules the whole project follows. Follow them in what you add and your system will fit in with the rest of it.

---

## 1. Money goes through one function

`BP_StoreManager.AddTransaction(Amount, Reason, ProductId)` is the only place the balance changes. Call it, with a reason from `E_TransactionReason`, and your cost or income shows up in the daily report on its own. See [Money and the store rating](../store/money_and_rating.md).

## 2. Subscribe to the clock, do not start a timer

`BP_StoreManager` holds the day and the hour and broadcasts `OnHourChanged` and `OnDayEnded`. Subscribe to those, and freezing the clock at closing time freezes your system too, with nothing written to make that happen.

World timers also do not run while the game is paused. Anything a paused screen depends on has to run from a widget `Tick`.

## 3. Reuse the storage component

Shelves, the back room rack and delivery boxes all store things with `BP_StorageComponent` and `BP_StorageRow`. If your system holds goods, use them rather than writing a second stock system. See [How shelves and storage work](../stock/how_storage_works.md).

## 4. AI decisions go in the brain, not in the tree

`UpdateTarget()` on the customer brain is the only thing that sets where a customer goes and what they look at, and the employee brain works the same way. A behaviour tree task writes a value into the brain and lets the brain decide; it never sets a destination or a focus itself.

## 5. Give employees work through the task queue

Publish a task, and a free employee qualified for it will take it. That is how you add a job without touching any existing AI. See [How employees work](../staff/how_employees_work.md).

## 6. Camera moves go through the view component

`BP_ViewMotionComponent` is the only thing that writes the view root's position and the camera's rotation. Add an input to it rather than calling `SetRelativeRotation` on the camera, and head bob, breathing and sway keep working on top of your move.

---

## Two habits

**Find out which native system already owns the value.** The character movement component owns velocity and rotation, the AI controller owns focus, the nav mesh owns pathing, the attachment graph keeps a widget beside an object. Drive it with its own flags rather than writing the value every frame.

**A state that persists gets one owner that re-asserts it**, not several places cleaning up after each other. Decide who owns it when you write the system, not when the bug appears.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
