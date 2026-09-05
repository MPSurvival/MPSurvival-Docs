# How a day works

A day in Grand Opening is a segment, not a loop. The clock starts at opening time, runs forward, and then **stops**. It never rolls over on its own. The next day only starts when you decide it does.

That decision is the OPEN sign by the door.

---

## The clock

`BP_StoreManager` owns the day number and the hour, and nothing else in the project moves time forward. It broadcasts `OnHourChanged` every game hour and `OnDayEnded` when a day is closed, and everything that cares subscribes to those two.

Its settings live in `DA_StoreConfig`:

| Field | What it does | Shipped default |
|---|---|---|
| `OpeningHour` | The hour a new day starts on | `8` |
| `ClockStopHour` | The hour the clock freezes at | `22` |
| `HourDurationSeconds` | Real seconds per game hour | `60` |
| `StartingBalance` | Money on day 1 | |
| `StartingRating` | Store rating on day 1 | |
| `RatingInertia` | How fast the rating chases the day's satisfaction | `0.15` |

At the shipped values a day is **14 real minutes** long. The sun sets at 18:00, so the last four hours are played at dusk and then at night with the street lamps on. See [The day/night cycle](../look/day_night_cycle.md).

`ClockStopHour` is not a closing time. Nothing shuts at 22:00, customers keep coming, sales keep happening. The clock simply stops counting, which gives you as long as you want to finish serving the queue, tidy up and walk to the sign.

---

## Closing the store

1. Interact with the **OPEN sign** by the front door.
2. A modal asks you to confirm.
3. The day report animates in, line by line.
4. `START A NEW DAY` fades the screen to black, cleans the world and starts the next day.

The sign refuses while a customer is still shopping. It polls twice a second and disables itself unless every customer left in the level is in `Leaving` or `LeavingAngry`, so you cannot close on someone standing at your till with a full basket. When it refuses, the outline and the prompt go away entirely rather than showing a prompt that does nothing.

---

## What gets reset between two days

`BP_StoreGameMode.CleanForNextDay()` is three calls, in this order, and it is the extension point for anything you add:

| Step | What it does |
|---|---|
| `DespawnCustomers` | Destroys every customer in the level |
| `ResetCheckouts` | Clears the belt, the receipt, the basket, the till state and the queue on every checkout |
| `MovePlayerToStart` | Teleports you back to the `PlayerStart` and points the camera the right way |

Then `FinishCurrentDay` runs: it fires `OnDayEnded` (which is what charges rent, drifts the market, pays wages and re-evaluates unlocks), moves the day counter on, and sets the clock back to `OpeningHour`.

If you add a system that has to start each day clean, put its call in `CleanForNextDay` rather than subscribing to something. That function is deliberately a flat list of calls so it stays readable.

**The world is not wiped.** Boxes you left on the floor are still there, shelves keep their stock, placed furniture stays placed. Only the customers, the checkout state and your own position reset.

---

## The daily report

`GetDayReport(Day)` builds an `S_DayReport` by reading the transaction journal, and that is the only place the numbers come from. Nothing is counted twice or kept in a parallel total.

The report reads in four blocks:

| Block | Lines |
|---|---|
| Customers | Served, Lost, Stock-outs, Total Customers |
| Store | Rating out of 5 |
| Revenue | Revenue, then Purchases, Wages, Rent, Utilities, Tax and Marketing |
| Result | Expenses, Total Profit, Balance |

**Total Customers is served plus lost**, not the sum of the four lines above it. A stock-out counts a customer who will also be counted as served or lost, so adding all four would double them.

Every line is a `BP_ReportRowWidget` dropped in the Designer, and the reveal cascade follows the order they sit in. Adding a line to the report means adding a widget in the Designer, not editing a graph.

---

## The HUD while the day runs

Top right, one panel: the store rating as a coloured ring, the balance in large type, and the day and clock on a secondary line reading `DAY 1 · MON` and `8:00 AM`.

Top left, notifications stack up and fade out. They are posted for things you would otherwise miss: a customer waiting at an unattended checkout, the store rating crossing a whole number in either direction.

See [The HUD and the prompt](../ui/hud_and_prompts.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
