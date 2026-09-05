# Money and the store rating

---

## Changing the balance

One function moves money: `AddTransaction(Amount, Reason, ProductId)` on `BP_StoreManager`.

Money in is positive, money out is negative. Every call is written to the transaction journal with a reason from `E_TransactionReason`, and the daily report is read back from that journal.

| Reason | Written by |
|---|---|
| `Sale` | the checkout, when a sale is finished |
| `Purchase` | the Market app, one row per cart line |
| `Rent` | the economy manager, at day end |
| `Utilities` | the economy manager, at day end |
| `Salary` | the staff manager, at day end |
| `Tax` | the economy manager, at day end |
| `Marketing` | the ad manager, per campaign per day |
| `Refund` | yours to call |
| `Fine` | yours to call |

**To add a cost or an income of your own**, and get its line in the daily report:

1. Add a value to `E_TransactionReason`.
2. Call `AddTransaction` with it from wherever the money moves.
3. Add a `BP_ReportRowWidget` for it in the report, in the Designer.

---

## The rating

A float from 0 to 5, shown as the ring in the top right of the HUD. Three things move it:

| What | Effect |
|---|---|
| A customer paid and left happy | up |
| A customer ran out of patience and left angry | down |
| A customer could not find what they came for | down |

A running ad campaign adds its `RatingDelta` on top, and the result is clamped to 0 - 5.

The rating eases toward the day's satisfaction rather than jumping to it. The speed is `RatingInertia` on `DA_StoreConfig`: `0.15` ships, which means one bad day dents the rating without erasing a good week.

**To add a fourth thing that moves the rating**, call `RecomputeRating` on `BP_StoreManager` from wherever it happens.

The rating gates content: unlocks read it as a condition, so `DA_Unlock_ColdChain` needs a rating of 3. See [Unlocks](../progress/unlocks.md).

**To change its colours**, open `MI_RatingRing`: four swatches drive the ramp, red at 0 through green at 5.

---

## Cash sales record what changed hands

A cash sale writes what you actually took, minus the change you actually gave back, not the total on the receipt. Hand back too much and you are down by the difference; hand back too little and you keep it. Counting change badly costs money on its own, with no penalty rule anywhere.

See [Cash, change and the card terminal](../checkout/payment_and_change.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
