# Rent, tax and the wholesale market

`BP_EconomyManager` runs once a day, on `OnDayEnded`, and it does two things: it charges you, and it moves wholesale prices.

Its settings are one asset, `DA_EconomyConfig`.

| Field | What it does | Shipped default |
|---|---|---|
| `DailyRent` | Charged every day, reason `Rent` | `75` |
| `DailyUtilities` | Charged every day, reason `Utilities` | `25` |
| `TaxRate` | Fraction of the day's sales revenue, reason `Tax` | `0.05` |
| `MarketVolatility` | How far a quote drifts per day | |
| `MarketRange` | How far a quote can get from its base price | |
| `DemandAmplitude` | How much ordering a product pushes its price up | |

The tax is computed on the revenue of the day that just ended, read from the same `GetDayReport` the report widget uses. It is charged **after** the day is closed, so it always taxes a finished number.

A day with no sales still writes a `Tax` line at `0.00`. One transaction per reason per day is regular and reads cleanly, and hiding the zero would have cost a branch.

---

## The market

Every product in the Market app's catalogue carries a **quote**, an `S_MarketQuote` holding the product and a multiplier that starts at `1.0`.

The wholesale price of a product is:

```
GetWholesalePrice(Product) = Product.CostPrice × quote multiplier
```

That is the only place a purchase price is read anywhere in the project. The Market app calls it for the unit price, multiplies by `UnitsPerCase` for the case price, and that same number is what leaves your balance when you order.

The multiplier is a multiplier and not a copied price on purpose. Change `CostPrice` on a `DA_Product_*` and the quote moves with it in the same proportion, and a product with no quote at all simply sells at its catalogue price.

---

## How a quote drifts

`DriftMarket(Day)` runs once per day, one line per quote:

- If you **ordered** that product today, the quote goes **up** by `DemandAmplitude`.
- If you did not, it drifts back toward `1.0` by the same amount.

Then the whole thing wanders inside `MarketRange` at `MarketVolatility` per day.

The result is that stocking up on one product every single day gets steadily more expensive, and leaving it alone brings it back to normal. Buying in bulk on a quiet day is worth something, and that is the whole point of the system.

Whether you ordered a product is read from the transaction journal, not from a counter kept on the side: `HasTransaction(Day, Purchase, ProductId)`.

---

## What the market does not show

The Market app prints the price of the day. It does not print the difference from the catalogue price, or an arrow, or a percentage. You see prices move between two days if you were paying attention, and not otherwise.

A `+8%` badge next to the unit price is about twenty minutes of widget work if you want it. `UnitCostOf(Product)` in `BP_MarketAppWidget` already has both numbers in hand.

---

## Wages

Wages are not the economy manager's job, they belong to `BP_StaffManager`. It pays the sum of the `DailyWage` of everyone on the roster who worked that day, at `OnDayEnded`, with reason `Salary`. See [How employees work](../staff/how_employees_work.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
