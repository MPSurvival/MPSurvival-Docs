# How a day works

The clock starts at opening time, runs forward and then freezes. It never rolls over on its own: the next day starts when **you** close the store, on the OPEN sign by the front door.

---

## The settings

One asset holds the whole day: `Content/GrandOpening/Blueprints/DataAssets/Store/DA_StoreConfig`.

| Field | What it does | Shipped |
|---|---|---|
| `OpeningHour` | The hour a new day starts on | `8` |
| `ClockStopHour` | The hour the clock freezes at | `22` |
| `HourDurationSeconds` | Real seconds per game hour | `60` |
| `StartingBalance` | Money on day 1 | |
| `StartingRating` | Store rating on day 1 | |
| `RatingInertia` | How fast the rating follows the day's satisfaction. `0` never moves, `1` snaps | `0.15` |

At the shipped values a day lasts **14 real minutes**. For longer days, raise `HourDurationSeconds`. For more hours in a day, raise `ClockStopHour`.

`ClockStopHour` is not a closing time. Nothing shuts at 22:00, customers keep coming and sales keep happening. The clock only stops counting, which gives you as long as you want to finish the queue and tidy up.

---

## Closing the store

1. Interact with the **OPEN sign** by the front door.
2. Confirm on the modal.
3. The day report animates in, line by line.
4. `START A NEW DAY` fades to black and starts the next day.

The sign refuses while a customer is still shopping, so you cannot close on someone standing at your till. When it refuses, its outline and prompt disappear entirely.

---

## What resets between two days

| Reset | Kept |
|---|---|
| Every customer in the level | Boxes left on the floor |
| The belt, receipt, basket, till and queue of every checkout | Stock on the shelves |
| Your own position, back at the `PlayerStart` | Furniture you placed, your balance, your rating |

If you add a system that has to start each day clean, add its call to `CleanForNextDay` in `BP_StoreGameMode`. That function is a flat list of calls, so a new line is all it takes.

---

## The daily report

The report reads in four blocks:

| Block | Lines |
|---|---|
| Customers | Served, Lost, Stock-outs, Total Customers |
| Store | Rating out of 5 |
| Revenue | Revenue, then Purchases, Wages, Rent, Utilities, Tax and Marketing |
| Result | Expenses, Total Profit, Balance |

`Total Customers` is Served plus Lost. A stock-out is a customer who also shows up in one of those two, so the four lines are not meant to be added together.

**To add a line**, drop another `BP_ReportRowWidget` into the report in the Designer. The reveal cascade follows the order the widgets sit in. No graph to edit.

---

## The HUD while the day runs

Top right: the store rating as a coloured ring, the balance in large type, and the day and clock underneath. Top left: notifications, posted when a customer is waiting at an unattended checkout or when the rating crosses a whole number.

See [The HUD and the prompt](../ui/hud_and_prompts.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
