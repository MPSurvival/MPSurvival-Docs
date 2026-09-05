# Money and the store rating

Two numbers drive the whole game, and each of them has exactly one writer.

---

## Money: one function, no exceptions

```
BP_StoreManager.AddTransaction(Amount, Reason, ProductId)
```

That is the only place in the project where the balance changes. A sale, a purchase, rent, a wage, a tax, an ad campaign: all of them call it. Money in is positive, money out is negative.

Every call is appended to a journal of `S_Transaction` rows, and the reason comes from `E_TransactionReason`:

| Reason | Written by |
|---|---|
| `Sale` | the checkout, when a sale is finished |
| `Purchase` | the Market app, one row per cart line |
| `Rent` | the economy manager, at day end |
| `Utilities` | the economy manager, at day end |
| `Salary` | the staff manager, at day end |
| `Tax` | the economy manager, at day end |
| `Marketing` | the ad manager, per campaign per day |
| `Refund` | nothing yet |
| `Fine` | nothing yet |

The last two are declared and unused. They are there for you to hook up.

Because everything goes through one function with a reason attached, the daily report is **derived** from the journal rather than accumulated alongside it. `SumForDay(Day, Reason)` gives you one line of the report; `CountForDay` and `HasTransaction(Day, Reason, ProductId)` answer the two other questions the game asks of the journal.

Add a cost of your own and you get its report line for free: add a value to `E_TransactionReason`, call `AddTransaction` with it, and add the row to the report widget.

The balance broadcasts `OnBalanceChanged`. The HUD, the computer's title bar and every app subscribe to it, none of them poll.

---

## The rating

The store rating is a float from 0 to 5, and `RecomputeRating()` is the only function that writes it.

Three things feed it today, each from exactly one caller:

| What | Where it is reported |
|---|---|
| A customer paid and left happy | `BP_CheckoutBase.FinishSale` |
| A customer ran out of patience and left angry | `BP_CustomerBrain.LeaveAngry` |
| A customer could not find anything they wanted | `BP_CustomerBrain.ReportStockOut` |

Running ad campaigns add their `RatingDelta` on top, and the result is clamped to 0 to 5.

The rating does not jump to the day's satisfaction, it **eases** toward it. `RatingInertia` on `DA_StoreConfig` is the speed: at `0` the rating never moves, at `1` it snaps to the day's value. `0.15` is the shipped value, which means a bad day dents the rating without erasing a good week.

Adding a fourth factor is one call into `RecomputeRating`. Do not write the rating from anywhere else, or the number stops meaning anything the moment two systems disagree.

---

## What the rating is worth

It is a metric the unlock system reads, so it gates content. `DA_Unlock_ColdChain` needs a rating of 3, for example. See [Unlocks](../progress/unlocks.md).

The HUD ring shows it, and its colour is a ramp inside `MI_RatingRing`: red at 0, orange around 1.7, yellow around 3.3, green at 5. A single `Fill` input drives both the arc and the colour, so they can never disagree. Changing the mood of the rating display is four colour swatches in that material instance.

When the rating crosses a whole number in either direction, a notification says so.

---

## Money never lies about what happened

One detail that is worth understanding, because it is the reason cash handling is a game and not a formality.

`FinishSale` does not record the total of the receipt. It records **what actually changed hands**: `AmountGiven − ComposedChange` for a cash sale, and whatever you typed into the card terminal for a card sale. Give too much change back and you lose the difference. Give too little and you keep it.

There is no separate penalty rule for miscounting. The journal just records the truth, and the truth costs money.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
