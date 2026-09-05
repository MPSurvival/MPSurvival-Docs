# Add a customer archetype

An archetype is one Data Asset, and it changes how a customer actually behaves rather than just how fast they walk.

Three ship with the template, and they are deliberately far apart:

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
4. Add it to `ArchetypeMix` on `DA_DayConfig` with a weight, or it will never appear.

---

## The fields

| Field | What it does |
|---|---|
| `WalkSpeed` | Movement speed, applied at spawn |
| `PatienceSeconds` | How long they will queue before leaving angry |
| `BudgetMin` / `BudgetMax` | The wallet, rolled per customer between the two |
| `TargetItemsMin` / `TargetItemsMax` | How many items they intend to buy, rolled per customer |
| `PreferredCategories` | Which `E_ProductCategory` values they will buy at all |
| `PriceSensitivity` | Declared, and nothing reads it yet |
| `BrowseSeconds` | How long they stand in front of a shelf before moving on |
| `CosmeticSet` | A `DA_CosmeticSet_*`, the wardrobe they are dressed from |

Speed, budget and item count are rolled per customer inside the archetype's bounds, so two hurried shoppers are not identical.

`PriceSensitivity` has nothing to bite on because selling prices are fixed on the product and there is no in-game pricing screen. It is left in place for the day one exists rather than being wired to a constant.

---

## Weighting the mix

`DA_DayConfig.ArchetypeMix` is a list of `S_ArchetypeMix` rows: an archetype and a weight. Each spawn draws from it.

Weights are relative, so `3 / 2 / 1` and `30 / 20 / 10` do the same thing. An archetype at weight `0` never appears, which is a useful way of switching one off without deleting it.

---

## Dressing them

`CosmeticSet` points at a `BP_CosmeticSetDataAsset`, which is a list of slots. Each slot is a `BP_CosmeticSlotDataAsset` holding a list of skeletal meshes for one part of the body.

Two slots ship: `DA_CosmeticSlot_Hair` and `DA_CosmeticSlot_Torso`.

At spawn, the cosmetic component creates one skeletal mesh component per slot and picks one option at random. **An empty entry in `Options` means that slot can be nothing**, which is how you get some customers with hats and some without. There is no probability field, the draw is enough.

Because the wardrobe is chosen on the archetype, two customer types can dress from two different sets.

Adding a body part is one more `DA_CosmeticSlot_*` in the set: legs, a bag, glasses. Nothing else changes.

If you add cosmetic meshes of your own, set their collision to none. A skeletal mesh created at runtime arrives with default collision, and hair that blocks the world will block the interaction trace and the customers behind it.

---

## Overriding the buying rule

`Wants(Product, Budget) → Wanted` lives on the archetype Data Asset, not on the brain. The brain just delegates to it.

That means you can subclass `BP_CustomerArchetypeDataAsset`, override `Wants`, and get a customer type with a genuinely different shopping rule without touching the AI. A customer who only buys what is on ad, or who refuses anything above a certain price, is one overridden function.

---

## Testing one archetype on its own

Put a single row in `ArchetypeMix` with your archetype at weight 1, set `CustomersPerHour` to something generous for the current hour, and play. Everyone who walks in is yours.

Do not place a customer by hand in the level to test. A hand-placed customer never goes through `SpawnCustomer`, so it never gets initialised, and it will behave as though it has no archetype at all.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
