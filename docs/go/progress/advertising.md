# Ad campaigns and sidewalk panels

A campaign is a poster and three effects. You start it from the **Ads** app, and it only works while there is an easel out front to show it on.

**One easel shows one campaign, in the order they were started.** A campaign beyond the number of easels is paused: no effect, no billing, and its day counter stops. Put another easel down and it comes back; pick one up and the last campaign pauses. There is nothing to assign.

---

## Adding a campaign

1. Draw a poster and import it.
2. Create a `DA_Ad_<Name>` from `BP_AdCampaignDataAsset` in `Blueprints/DataAssets/Ads/Childs/` and fill it in.
3. Add it to `AvailableCampaigns` on `DA_App_Ads`.

The app row, the billing, the effects and the poster on the easel all come from the asset.

Keep **prices and percentages off the poster**: a campaign brings customers through the door, and what they pay is set on the product asset, so a poster promising a cut price would advertise something the shelf does not do.

---

## The fields

| Field | What it does |
|---|---|
| `DisplayName` | The name in the app |
| `Poster` | The texture shown on the easel |
| `CostPerDay` | Billed at the start, then every day |
| `DurationDays` | How many days it runs for |
| `TrafficMultiplier` | Multiplies the footfall from the day config |
| `PromotedCategories` | Categories customers will buy even outside their preferences |
| `RatingDelta` | Added to the store rating while it runs |

Three ship with the template:

| Campaign | Cost/day | Days | Effect |
|---|---|---|---|
| `DA_Ad_FlyerDrop` | 15 | 3 | ×1.15 traffic |
| `DA_Ad_FreshDeals` | 40 | 5 | ×1.25 traffic, pushes `Fresh` and `Food` |
| `DA_Ad_LoudSale` | 25 | 4 | traffic and rating |

---

## What the three effects do

**Traffic** multiplies the hourly figure from `DA_DayConfig`. Two campaigns at ×1.15 and ×1.25 give ×1.44.

**Rating** adds every live `RatingDelta` to the target rating, clamped to 0 - 5.

**Categories** are read **when a customer spawns**. Someone who walks in under a Fresh Deals campaign buys fresh food even if their archetype normally would not, and keeps that for the whole visit even if you stop the campaign while they are in the shop.

---

## Running one

Each row in the **Ads** app shows the poster, the name, the traffic and rating effects, the cost per day, the days left, and a `RUN` or `STOP` button.

`RUN` needs a free easel, a campaign that is not already running, and enough money for one day. The first day is billed on start and the rest at the end of each day, so it gets its own `Marketing` line in the daily report.

---

## The easel

`BP_AdPanel` is bought from the **Structures** app like any other furniture and placed with `B`. It is allowed in `SalesFloor` and `Outside`, so it goes in the window or on the pavement.

It is not interactable: the campaign is chosen in the app, where you can see what it costs.

The poster is drawn on the material slot named `M_AdPoster`. `BlankPoster` is what shows with no campaign — leave it filled in, because an empty texture parameter falls back to the master material's wood grain.

---

## Before you test it

The Ads app and the easel are both rewards on `DA_Unlock_Marketing`, which needs day 3. Before day 3 neither of them exists, which is the thing most likely to make you think the system is broken on a first run.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
