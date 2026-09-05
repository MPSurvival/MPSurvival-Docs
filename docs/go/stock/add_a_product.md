# Add a product

A product is one Data Asset. There is no graph to open, no list to register it in, and no ID to keep unique by hand.

Six ship with the template, in `Blueprints/DataAssets/Products/Childs/`: a cereal box, a can of soup, a soda can, a shampoo bottle, laundry detergent and a battery pack. They are deliberately different sizes, because size is what drives the shelf layout.

---

## The recipe

1. Right click in `Blueprints/DataAssets/Products/Childs/` → **Miscellaneous → Data Asset** → pick `BP_ProductDataAsset`.
2. Name it `DA_Product_<Thing>`.
3. Fill it in.
4. Add it to `DA_App_Market.AvailableProducts` so it can be ordered.

That last step is the only one people forget. A product that is not in the Market app's catalogue exists, works on a shelf and can be scanned, but nothing can ever order it.

---

## The fields

| Field | What it does |
|---|---|
| `DisplayName` | The name shown on the market row, the receipt and the checkout screen. It is a `Text`, so it can be translated |
| `Category` | One of `E_ProductCategory`. This is what shelves accept or refuse |
| `Mesh` | The static mesh, used on shelves, in boxes, on the belt and in flight |
| `Icon` | The thumbnail, used on the market row, the receipt, the box label and the carry panel |
| `CostPrice` | What you pay per unit. The market quote multiplies this |
| `SuggestedPrice` | What the customer pays per unit |
| `FootprintCm` | X is the width along a row, Y is the depth into it |
| `StackHeightCm` | The height of one unit, used for stacking |
| `StackMax` | How many can be stacked on top of each other |
| `UnitsPerCase` | How many units come in one ordered case |
| `DeliveryDelayHours` | Hours between ordering and the box arriving |
| `DeliversNextDay` | If ticked, the box arrives at opening time the next day instead, whatever the hour |

`DeliversNextDay` wins over `DeliveryDelayHours` when both are set.

`RequiresRefrigeration` is also on the asset and nothing reads it yet. The fridge restricts what it accepts by category like any other shelf, through its `DA_ShelfType_FridgeTwoDoor`.

---

## Getting the footprint right

`FootprintCm` and `StackHeightCm` are not cosmetic. They decide how many units fit on a shelf and how many fit in a cardboard box, and both are computed at runtime from these numbers.

Measure the mesh rather than guessing:

1. Open the static mesh.
2. Read the **Bounds** in the details panel, or use the top and side orthographic views.
3. `FootprintCm.X` is the width, `FootprintCm.Y` is the depth, `StackHeightCm` is the height.

Add a millimetre or two if you want visible gaps between units on a shelf. Take one away and units interpenetrate.

The consequences are immediate and worth knowing:

- A wide product means fewer facings per row.
- A tall product means a row with low clearance refuses it, and it ends up on the top shelf.
- A large product means fewer units per case, because the box capacity is derived from the same numbers.

---

## Pricing

`CostPrice` is your buying price per unit and `SuggestedPrice` is the selling price. The margin between them is the whole business.

A case costs `CostPrice × UnitsPerCase × market quote`, and that is what the Market app shows in its `TOTAL` column and what leaves your balance. A sale brings in `SuggestedPrice` per unit.

The selling price is fixed on the asset. There is no in-game price setting screen, so `PriceSensitivity` on the customer archetypes has nothing to bite on yet and is left unread on purpose.

---

## Where a new product shows up on its own

Once the asset exists and it is in `DA_App_Market.AvailableProducts`:

- It appears in the Market app, with its icon, price and case size.
- Ordering it spawns a cardboard box sized to fit it, with the right number of units inside and its icon printed on the docket.
- It can be put on any shelf whose type accepts its category.
- Customers whose archetype prefers that category will come looking for it.
- It scans at the checkout, groups on the receipt and shows up in the transaction journal.

None of that needs a line of Blueprint.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
