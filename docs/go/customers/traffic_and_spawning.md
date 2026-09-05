# Traffic, and when a customer is born

Footfall is a curve you fill in by hand, hour by hour, and two gates that can stop a spawn from happening at all.

---

## The day config

`DA_DayConfig` is one asset with three fields.

| Field | What it does |
|---|---|
| `CustomersPerHour` | 24 entries, indexed by hour. How many customers arrive during that hour |
| `ArchetypeMix` | The weighted list of archetypes to draw from |
| `DifficultyPerDay` | A ramp applied as the days go by |

`CustomersPerHour` is the shape of your day. Index 8 is 08:00. Put a bump at lunchtime and a bigger one at six in the evening and the shop breathes.

Entries for hours the store is not open do not matter, because the clock never reaches them.

---

## How the spawner uses it

`BP_CustomerSpawner` re-arms itself:

```
delay = HourDurationSeconds / CustomersPerHour[current hour]
```

It uses an engine timer, not a tick, and it re-plans on `OnHourChanged` so it follows the curve as the day moves. It also re-plans **before** deciding whether to spawn, which means a refused spawn never stops the spawner. It will keep trying and start producing again the moment the store is worth visiting.

Ad campaigns multiply this rate. `GetCustomerRate` takes the value from the day config and multiplies it by every live campaign's `TrafficMultiplier`. See [Ad campaigns](../progress/advertising.md).

---

## The two gates

A customer is not born if either of these is false. Both exist for the same reason: a customer who walks in, finds nothing, and leaves angry is worse for the player than no customer at all.

**1. There has to be something they would buy.**

The spawner checks the archetype it drew against what is actually on the shelves. Nothing in stock that this type wants and could afford, no spawn.

The test uses `BudgetMax` rather than the customer's real budget, because the real budget is rolled during initialisation, which happens after the spawn. So the question is "could this *type* of customer want something here", not "can this exact person afford it".

**2. There has to be a usable checkout.**

At least one checkout that is `Open` **and** has a `PairedScanner`. A shop with no way to take money produces no customers, exactly like a shop with nothing on the shelves.

This gate is structural, not situational. It asks whether a usable till exists, not whether a queue has space. A full queue is a manageable problem and the customer will wait.

---

## Where they come from and where they go

| Actor | What it does |
|---|---|
| `BP_CustomerSpawner` | Where customers appear. Set `CustomerClass` on it |
| `BP_CustomerExit` | Where they walk to when they leave |

The brain resolves the exit once at spawn, with `Get Actor Of Class`. There is no hard asset path anywhere.

Both actors have to be **inside** the `NavMeshBoundsVolume`. `Move To` will not project a destination outside it, and a customer sent to an unreachable exit stands still forever rather than complaining.

---

## Navigation settings that matter

Two settings that will bite you in a store layout, because a store is aisles.

| Setting | Value | Why |
|---|---|---|
| `RecastNavMesh` → Cell Size | `10` | It has to be at most a third of the narrowest aisle you want walkable |
| Agent Radius | equal to the character capsule radius | An agent wider than the capsule refuses aisles the character fits through |

Runtime generation is set to **Dynamic** in `DefaultEngine.ini`, which is what lets a gondola placed during a trading day be walked around immediately.

Anything that moves to open a passage, a door leaf or a checkout gate, needs **Is Dynamic Obstacle** ticked on the **static mesh asset**, not on the component.

A carried box has to stop carving a hole in the nav mesh. Navigation relevance follows whether the collision blocks `Pawn` or `Vehicle`, so a prop that blocks pawns will keep affecting the nav mesh even while it is in your hands.

---

## Difficulty over time

`DifficultyPerDay` ramps the pressure as the days go by. It is the field to reach for if you want day 20 to be busier than day 2 without editing the hourly curve.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
