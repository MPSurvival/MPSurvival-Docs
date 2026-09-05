# Unlocks

Progression gates content behind metrics. Reach a rating, a balance, a day or a customer count, and a set of assets becomes available, either free or for a price.

The key of an unlock is **the Data Asset itself**, never an identifier string.

---

## Why that matters

Products, structures, apps, campaigns and employee candidates all derive from `PrimaryDataAsset`. So a single field, `Rewards`, typed as an array of `PrimaryDataAsset`, covers all five, and it covers whatever kind of asset you add later without the manager changing.

An identifier would have meant a lookup table per family, and it would have covered none of what you write yourself.

---

## The Data Asset

`BP_UnlockDataAsset`, one instance per unlock.

| Field | What it does |
|---|---|
| `DisplayName` | The name in the Upgrades app |
| `Description` | The line under it |
| `Icon` | The thumbnail |
| `Requirements` | A list of `S_UnlockRequirement`: a metric and a value. **All** must be met |
| `Requires` | Other unlocks that must be granted first |
| `Rewards` | The assets this unlock releases |
| `Price` | See below |

`Requires` is a typed field rather than a metric, because an enum and a float cannot name an asset.

---

## The five metrics

`E_UnlockCondition` holds what the manager can read without anyone adding a counter:

| Metric | Read from |
|---|---|
| `Money` | The current balance |
| `TotalRevenue` | The journal, filtered to `Sale` |
| `Rating` | The store rating |
| `Day` | The day number |
| `CustomersServed` | The per-day counter |

`GetMetric(Condition)` is a single switch and the only place a metric is read. Adding a sixth is an enum entry and a branch.

There is no `UnitsSold`, because `S_Transaction` does not carry a quantity, and adding one would have meant changing the checkout to satisfy a progression goal.

---

## Free or paid, decided by the data

`Price` picks the model:

| `Price` | Behaviour |
|---|---|
| `0` | Meeting the conditions grants it on its own |
| `> 0` | Meeting the conditions makes it **available**, and you buy it in the Upgrades app |

One field, two progression systems, no branch anywhere else.

---

## What "unlocked" means

```
IsUnlocked(Asset)  =  granted somewhere  OR  not mentioned by any unlock
```

**An asset nobody mentions is available.** Adding a product does not require writing an unlock for it. That is what keeps the system opt-in instead of turning into paperwork.

Five lists filter through it: the Market, Structures, Hiring and Ads apps, and the desktop itself. Placement follows for free, because you can only place what you own.

---

## When it is evaluated

`Evaluate` runs at the end of each day, and again when the Upgrades app is opened.

Not on every transaction. Three more subscriptions for something you would see in the evening report anyway was not worth it.

A granted unlock records **the day it was granted**, not just the fact. Without the date, an unlock earned at rating 3.0 and then read back at rating 2.4 would show `RATING 2.4 / 3` and `UNLOCKED` on the same line, which reads like a bug. With it, the line says `UNLOCKED (on day 2)`.

---

## The six shipped unlocks

Chosen so all five metrics, the prerequisite field and both price models are visible.

| Unlock | Gate |
|---|---|
| `DA_Unlock_ColdChain` | Rating |
| `DA_Unlock_Marketing` | Day 3. Releases the Ads app and the easel |
| `DA_Unlock_Electronics` | Revenue |
| `DA_Unlock_Boutique` | Money, and a prerequisite |
| `DA_Unlock_Comfort` | Customers served |
| `DA_Unlock_SeniorStaff` | Releases a hiring candidate |

---

## The Upgrades app

One row per unlock: thumbnail, name, description underneath, a progress column, a price, and a button.

The progress column names the metric and shows both numbers: `RATING 3.2 / 4.0`. The label comes from a switch on `E_UnlockCondition`, so a sixth metric needs a branch there too.

A free unlock shows no price and no button. There is nothing to click.

---

## Adding one

1. Create a `DA_Unlock_<Name>` from `BP_UnlockDataAsset`.
2. Put one or more `Requirements` in it.
3. Put the assets it releases in `Rewards`.
4. Set `Price` to `0` for automatic, or a number to make it purchasable.
5. Add it to `Catalog` on `BP_ProgressionManager`, in the Details panel of the game state.

Step 5 is the one that matters. An unlock that is not in the catalogue never gates anything, and its rewards stay available from the start.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
