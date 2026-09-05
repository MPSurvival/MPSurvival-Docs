# Ad campaigns and sidewalk panels

A campaign is a poster and three effects. It runs from the **Ads** app, and it only has an effect while there is an easel out front to show it on.

The rule, in one sentence: **one easel shows one campaign, in the order they were started.**

---

## How campaigns and panels line up

`BP_AdManager` keeps two lists: `Runs`, the campaigns that are running, and `Panels`, the easels placed in the world. Panel *i* shows campaign *i*, and the number of live campaigns is `min(Runs, Panels)`.

Everything else falls out of that:

- A campaign beyond the number of easels is **paused**. It has no effect, it is not billed, and its day counter does not move.
- Putting another easel down brings it back.
- Picking an easel up pauses the last campaign instead of orphaning it.

There is nothing to assign. That was the trade: you cannot choose which easel shows which poster, and in exchange there is no state to keep in sync and no way to end up with a campaign nobody can see.

---

## The Data Asset

`BP_AdCampaignDataAsset`, one instance per campaign.

| Field | What it does |
|---|---|
| `DisplayName` | The name in the app |
| `Poster` | The texture shown on the easel |
| `CostPerDay` | Billed at the start, then every day |
| `DurationDays` | How many days it runs for |
| `TrafficMultiplier` | Multiplies the footfall from the day config |
| `PromotedCategories` | Categories customers will buy even outside their preferences |
| `RatingDelta` | Added to the store rating while it runs |

Three ship with the template, deliberately different:

| Campaign | Cost/day | Days | Effect |
|---|---|---|---|
| `DA_Ad_FlyerDrop` | 15 | 3 | ×1.15 traffic |
| `DA_Ad_FreshDeals` | 40 | 5 | ×1.25 traffic, pushes `Fresh` and `Food` |
| `DA_Ad_LoudSale` | 25 | 4 | traffic and rating |

---

## The three effects

**Traffic** multiplies. `GetCustomerRate` takes the hourly figure from `DA_DayConfig` and multiplies it by every live campaign's `TrafficMultiplier`. Two campaigns running at ×1.15 and ×1.25 give ×1.44.

**Rating** adds. The sum of every live `RatingDelta` is added to the target rating and then clamped to 0 to 5. `RecomputeRating` is still the only thing that writes the rating.

**Categories** are read **once, when a customer spawns**. A customer who walks in under a Fresh Deals campaign will buy fresh food even if their archetype does not normally like it, and they keep that for their whole visit even if you stop the campaign while they are in the shop.

Reading it once rather than per query is a real decision: `IsProductWanted` is called per product per shelf, and rebuilding the promoted list every time would mean resolving the game state hundreds of times a minute for a value that changes once a day.

---

## Running one

In the **Ads** app, each row shows the poster, the name, the traffic and rating effects, the cost per day, the remaining days, and a `RUN` or `STOP` button.

`CanStartCampaign` is the single rule behind the button: there has to be a free easel, the campaign must not already be running, and you have to be able to afford one day.

The first day is billed on start, the rest at the end of each day, with reason `Marketing`. It goes through `AddTransaction` like everything else, so it shows up as its own line in the daily report.

---

## The easel

`BP_AdPanel` is bought from the **Structures** app like any other piece of furniture, through `DA_Structure_AdPanel`, and placed with the build key. It is allowed in `SalesFloor` and `Outside`, so it can go in the window or on the pavement.

It holds no state at all. It registers itself with the manager at `BeginPlay`, removes itself at `EndPlay`, and exposes `ShowCampaign`, which only the manager calls.

The poster is a dynamic material instance created once, on the material slot named `M_AdPoster`. `BlankPoster` is what it shows with no campaign, and it is a real protection rather than a nicety: setting a texture parameter to nothing falls back to the master material's default, which is a wood grain.

The easel itself is not interactable. A campaign is chosen in the app, where you can see what it costs and what it does. Pressing `E` on the panel would have nothing to say.

---

## Adding a campaign

1. Draw a poster and import it.
2. Create a `DA_Ad_<Name>` from `BP_AdCampaignDataAsset` and fill it in.
3. Add it to `AvailableCampaigns` on `DA_App_Ads`.

Nothing else. The app row, the billing, the effects and the poster on the easel all come from the asset.

One thing the shipped posters avoid, and yours probably should too: **no prices and no percentages.** There is no discount system in the template, so a poster reading "−50%" would be advertising something the game cannot do.

---

## Before you test it

The Ads app and the easel are both rewards on `DA_Unlock_Marketing`, which needs day 3. Before day 3 neither of them exists.

That is intended, and it is also the thing most likely to make you think the system is broken on your first run.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
