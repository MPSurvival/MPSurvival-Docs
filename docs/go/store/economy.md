# Rent, tax and the wholesale market

`BP_EconomyManager` runs once at the end of each day: it charges you, and it moves wholesale prices.

---

## The settings

One asset: `Content/GrandOpening/Blueprints/DataAssets/Store/DA_EconomyConfig`.

| Field | What it does | Shipped |
|---|---|---|
| `DailyRent` | Charged every day | `75` |
| `DailyUtilities` | Charged every day | `25` |
| `TaxRate` | Share of the day's sales revenue, charged after the day is closed | `0.05` |
| `MarketVolatility` | How far a quote drifts per day | |
| `MarketRange` | How far a quote can get from its base price | |
| `DemandAmplitude` | How much ordering a product pushes its price up | |

A day with no sales still writes a `Tax` line at `0.00`, so every reason has one row per day in the report.

---

## The wholesale market

Every product in the Market app carries a quote: a multiplier on its `CostPrice`, starting at `1.0`. What you pay for a case is `CostPrice` times that multiplier times `UnitsPerCase`.

Per day, for each product:

- Ordered it today, its price goes **up** by `DemandAmplitude`.
- Left it alone, its price drifts back toward its catalogue price.

On top of that every quote wanders inside `MarketRange`, at `MarketVolatility` per day.

So buying the same product every day gets steadily more expensive, and buying in bulk on a quiet day is worth something. The Market app shows the price of the day, with no arrow and no percentage.

**To change a product's base price**, edit `CostPrice` on its `DA_Product_*`. The quote is a multiplier, so the market price follows in the same proportion. See [Add a product](../stock/add_a_product.md).

---

## Wages

Wages belong to `BP_StaffManager`, not to the economy manager. It pays the `DailyWage` of everyone on the roster who worked that day, at day end. See [How employees work](../staff/how_employees_work.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
