# Add a customer archetype

An archetype is one Data Asset. It changes how a customer shops, not just how fast they walk.

Three ship with the template:

| Asset | Character |
|---|---|
| `DA_Customer_Hurried` | Fast, 240 cm/s, 25 seconds of patience, browses for 1.5 s, small basket |
| `DA_Customer_Regular` | The middle of the road |
| `DA_Customer_Bargain` | Price sensitivity 2.0, up to 10 items, browses longest |

---

## The recipe

1. Right click in `Blueprints/DataAssets/Customers/Childs/` → **Data Asset** → `BP_CustomerArchetypeDataAsset`.
2. Name it `DA_Customer_<Kind>`.
3. Fill it in.
4. Add it to `ArchetypeMix` on `DA_DayConfig` with a weight, or it never appears.

---

## The fields

| Field | What it does |
|---|---|
| `WalkSpeed` | Movement speed, applied at spawn |
| `PatienceSeconds` | How long they queue before leaving angry |
| `BudgetMin` / `BudgetMax` | The wallet, rolled per customer between the two |
| `TargetItemsMin` / `TargetItemsMax` | How many items they intend to buy, rolled per customer |
| `PreferredCategories` | Which `E_ProductCategory` values they will buy at all |
| `PriceSensitivity` | Declared on the archetype, for a pricing system of your own to read |
| `BrowseSeconds` | How long they stand in front of a shelf before moving on |
| `CosmeticSet` | A `DA_CosmeticSet_*`, the wardrobe they are dressed from |

Speed, budget and item count are rolled per customer inside those bounds, so two hurried shoppers are not identical.

---

## Weighting the mix

`ArchetypeMix` on `DA_DayConfig` is a list of archetypes and weights, drawn from at every spawn. Weights are relative, so `3 / 2 / 1` and `30 / 20 / 10` do the same thing. Weight `0` switches an archetype off without deleting it.

---

## Dressing them

`CosmeticSet` points at a `BP_CosmeticSetDataAsset`, which is a list of slots. Each slot is a `BP_CosmeticSlotDataAsset` holding the meshes available for one part of the body. Two ship: `DA_CosmeticSlot_Hair` and `DA_CosmeticSlot_Torso`.

One option per slot is drawn at random when a customer spawns. **An empty entry in `Options` means that slot can be nothing**, which is how some customers get a hat and some do not.

**To add a body part** — legs, a bag, glasses — add one more `DA_CosmeticSlot_*` to the set. Nothing else changes.

If you add cosmetic meshes of your own, **set their collision to none**. Hair that blocks the world blocks the interaction trace and the customers behind it.

---

## Overriding the buying rule

`Wants(Product, Budget)` lives on the archetype Data Asset. Subclass `BP_CustomerArchetypeDataAsset`, override it, and you have a customer type with a genuinely different shopping rule without touching the AI: one who only buys what is advertised, or refuses anything above a price.

---

## Testing one archetype on its own

Put a single row in `ArchetypeMix` with your archetype at weight 1, raise `CustomersPerHour` for the current hour, and play. Everyone who walks in is yours.

**Do not place a customer by hand in the level to test.** A hand-placed customer never goes through the spawner, so it is never given an archetype and behaves as if it had none.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
