# Make a loot item breakable

A breakable item shatters into pieces when it is hit hard enough, and its value is lost for good. It takes one pre-fractured mesh and six fields on the item's `DA_Loot_*` Data Asset. No graph to open.

Three items ship breakable: the glass vase, the ceramic urn and the small statue. If you have not read [How a run works](how_a_run_works.md), start there.

- The loot Data Assets: `Content/MPFriendslop/Blueprints/DataAssets/Loot/Childs/`
- The fractured meshes: `Content/MPFriendslop/Meshes/Props/Loot/Debris/`
- The pieces actor: `Content/MPFriendslop/Blueprints/Effects/BP_LootDebris`

---

## Before you start

You need a loot item that already works: a `DA_Loot_*` with a `Fragility` above `0`, used by a loot actor. An item with `Fragility` at `0` never loses value, so it can never break. See [Add a loot item](add_a_loot_item.md).

---

## Step 1, fracture the mesh

The pieces are a Chaos **Geometry Collection**, made once in the editor with Fracture Mode.

1. Drag the item's `SM_Loot_*` mesh into any level.
2. Select it, and switch the editor to **Fracture** mode.
3. Click **New**. Save the Geometry Collection as `GC_Loot_<Name>` in `Meshes/Props/Loot/Debris/`.
4. Pick **Uniform**. Set `Min Voronoi Sites` and `Max Voronoi Sites` to about `15` and `25`.
5. Click **Fracture**, then save.
6. Delete the actor from the level. Only the asset is needed.

---

## Step 2, fill the breaking fields

1. Open the item's `DA_Loot_*`.
2. In `Settings|Breaking`, tick `Can Break`.
3. Set `Debris Collection` to your `GC_Loot_<Name>`.
4. Set the two thresholds, the burst and the sound.
5. Save.

| Field | What it does | Shipped in `DA_Loot_GlassVase` |
|---|---|---|
| `Can Break` | Allows the item to shatter | ticked |
| `Break Single Hit Damage` | The item breaks when one hit takes away this share of its condition, from 0 to 1 | `0.4` |
| `Break Total Damage` | The item breaks when its total lost condition reaches this share, from 0 to 1 | `0.75` |
| `Debris Burst Strength` | How hard the pieces are blown apart | `6000` |
| `Break Sound` | Played where the item breaks | `CUE_Break_Glass` |
| `Debris Collection` | The fractured mesh from step 1. Leave it empty and the item simply disappears, with the sound but no pieces | `GC_Loot_GlassVase` |

The other two ship with `0.55` / `0.85` and a burst of `5000` on the urn, `0.18` / `0.6` and `3500` on the statue. Two sounds ship: `CUE_Break_Glass` and `CUE_Break_Metal`.

A hit only counts when it costs value, so the two thresholds work on top of the value fields you already set: `Fragility`, `Impact Speed Threshold` and `Reference Impact Speed`. A very fragile item with a low `Break Single Hit Damage` breaks on the first bad drop. A sturdy one with only `Break Total Damage` survives a few knocks and breaks on the last.

!!! warning
    One hit can never take away more than `Fragility`, so a `Break Single Hit Damage` above `Fragility` never breaks the item in one hit. The total lost condition can never pass `1`, so a `Break Total Damage` above `1` never breaks it at all. Nothing tells you: the item just takes damage and stays whole.

---

## The pieces

When the item breaks, every player's game spawns a `BP_LootDebris` at its place, fills it with the `Debris Collection`, and blows it apart. You change these on `BP_LootDebris` itself, in `Settings|Debris`:

| Field | What it does | Shipped |
|---|---|---|
| `Debris Life Span` | Seconds before the pieces are removed | `6` |
| `Burst Radius` | Radius of the push, in cm | `150` |

The push moves every Chaos physics body inside `Burst Radius`, not only the pieces, so loot lying next to a breaking vase gets shoved. Lower `Burst Radius` if that bothers you.

The pieces use the `LootDebris` collision profile, in the Project Settings. They land on the floor and furniture, but they never push a player, never block the camera and never answer the interaction trace.

To give one item its own pieces actor, make a child of `BP_LootDebris` and set it as `Debris Actor Class` on that item's `BP_LootValueComponent`. The template calls `Burst` on it, so it must stay a child of `BP_LootDebris`.

---

## Reacting to a break

Bind this on the item's `BP_LootValueComponent` to add your own effect, score or quest step:

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnBroken` | The item breaks, just before it is destroyed | every player |

A player holding the item when it breaks simply lets go. Loot lying on the extraction trap cannot break: the trap makes it immune to damage, and a hit that costs nothing never breaks anything.

---

Next, put your item in the map: [Place loot spawn points and loot tables](place_loot_spawn_points.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
