# Traffic, and when a customer is born

Footfall is a curve you fill in by hand, hour by hour, plus two conditions that can stop a spawn.

---

## The day config

`DA_DayConfig`, three fields:

| Field | What it does |
|---|---|
| `CustomersPerHour` | 24 entries, indexed by hour. How many customers arrive during that hour |
| `ArchetypeMix` | The weighted list of archetypes to draw from |
| `DifficultyPerDay` | A ramp applied as the days go by, to make day 20 busier than day 2 |

`CustomersPerHour` is the shape of your day: index 8 is 08:00. Put a bump at lunchtime and a bigger one at six in the evening and the shop breathes. Hours the store never reaches do not matter.

Ad campaigns multiply the rate on top of this. See [Ad campaigns](../progress/advertising.md).

---

## The two conditions

A customer is not born unless both are true. A customer who walks in, finds nothing and leaves angry is worse for the player than no customer at all.

**1. Something on the shelves this archetype would buy.** Nothing in stock that this type wants and could afford, no spawn.

**2. A usable checkout**: at least one that is `Open` and has a `PairedScanner`. A full queue is not a problem — the customer will wait — but a shop with no way to take money produces nobody.

A refused spawn never stops the spawner. It keeps trying and starts producing again the moment the store is worth visiting.

---

## Where they come from and where they go

| Actor | What it does |
|---|---|
| `BP_CustomerSpawner` | Where customers appear. Set `CustomerClass` on it |
| `BP_CustomerExit` | Where they walk to when they leave |

Both have to be **inside** the `NavMeshBoundsVolume`. A customer sent to an unreachable exit stands still forever.

---

## Navigation settings that matter in a store

| Setting | Value | Why |
|---|---|---|
| `RecastNavMesh` → Cell Size | `10` | At most a third of the narrowest aisle you want walkable |
| Agent Radius | The character capsule radius | A wider agent refuses aisles the character fits through |

Runtime generation is **Dynamic**, so a gondola placed during a trading day can be walked around immediately.

Anything that moves to open a passage — a door leaf, a checkout gate — needs **Is Dynamic Obstacle** ticked on the **static mesh asset**, not on the component.

If a prop you carry keeps carving a hole in the nav mesh, it is because its collision still blocks `Pawn` or `Vehicle`.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
