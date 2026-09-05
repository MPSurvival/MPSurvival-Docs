# Unlocks

Progression gates content behind metrics. Reach a rating, a balance, a day or a customer count, and a set of assets becomes available, either free or for a price.

---

## Adding one

1. Create a `DA_Unlock_<Name>` from `BP_UnlockDataAsset`.
2. Put one or more `Requirements` in it.
3. Put the assets it releases in `Rewards`.
4. Set `Price` to `0` for automatic, or a number to make it purchasable in the Upgrades app.
5. Add it to `Catalog` on `BP_ProgressionManager`, in the Details panel of the game state.

Step 5 is the one that matters. An unlock that is not in the catalogue never gates anything, and its rewards stay available from the start.

---

## The fields

| Field | What it does |
|---|---|
| `DisplayName` | The name in the Upgrades app |
| `Description` | The line under it |
| `Icon` | The thumbnail |
| `Requirements` | A list of metric-and-value pairs. **All** must be met |
| `Requires` | Other unlocks that must be granted first |
| `Rewards` | The assets this unlock releases |
| `Price` | `0` grants it as soon as the conditions are met; above `0` makes it available to buy |

`Rewards` takes **any** Data Asset: a product, a structure, an app, a campaign, a hiring candidate, or a kind of asset you add yourself.

---

## The five metrics

| Metric | Read from |
|---|---|
| `Money` | The current balance |
| `TotalRevenue` | Everything sold so far |
| `Rating` | The store rating |
| `Day` | The day number |
| `CustomersServed` | The per-day counter |

**To add a sixth**, add an entry to `E_UnlockCondition` and a branch to `GetMetric` on `BP_ProgressionManager`, plus a branch in the Upgrades app for its label.

---

## What "unlocked" means

**An asset nobody mentions is available.** Adding a product does not require writing an unlock for it, so the system stays opt-in.

Five lists filter through it: the Market, Structures, Hiring and Ads apps, and the desktop itself. Placement follows for free, since you can only place what you own.

Unlocks are evaluated at the end of each day, and again when the Upgrades app is opened. A granted unlock keeps the day it was granted on, so a line reads `UNLOCKED (on day 2)` even if the rating has since dropped back below the bar.

---

## The six shipped unlocks

| Unlock | Gate |
|---|---|
| `DA_Unlock_ColdChain` | Rating |
| `DA_Unlock_Marketing` | Day 3. Releases the Ads app and the easel |
| `DA_Unlock_Electronics` | Revenue |
| `DA_Unlock_Boutique` | Money, and a prerequisite |
| `DA_Unlock_Comfort` | Customers served |
| `DA_Unlock_SeniorStaff` | Releases a hiring candidate |

Between them they use all five metrics, the prerequisite field and both price models.

---

## The Upgrades app

One row per unlock: thumbnail, name, description, a progress column reading `RATING 3.2 / 4.0`, a price and a button. A free unlock shows no price and no button.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
