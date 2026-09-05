# Add a product

A product is one Data Asset. No graph to open, no list to register it in, no ID to keep unique by hand.

Six ship with the template, in `Blueprints/DataAssets/Products/Childs/`: a cereal box, a can of soup, a soda can, a shampoo bottle, laundry detergent and a battery pack.

---

## The recipe

1. Right click in `Blueprints/DataAssets/Products/Childs/` → **Miscellaneous → Data Asset** → pick `BP_ProductDataAsset`.
2. Name it `DA_Product_<Thing>`.
3. Fill it in.
4. Add it to `DA_App_Market.AvailableProducts` so it can be ordered.

Step 4 is the one people forget. A product that is not in the Market app's catalogue works on a shelf and scans at the till, but nothing can ever order it.

---

## The fields

| Field | What it does |
|---|---|
| `DisplayName` | The name on the market row, the receipt and the checkout screen |
| `Category` | One of `E_ProductCategory`. This is what shelves accept or refuse |
| `Mesh` | The static mesh, used on shelves, in boxes and on the belt |
| `Icon` | The thumbnail, used on the market row, the receipt, the box label and the carry panel |
| `CostPrice` | What you pay per unit |
| `SuggestedPrice` | What the customer pays per unit |
| `FootprintCm` | X is the width along a row, Y is the depth into it |
| `StackHeightCm` | The height of one unit |
| `StackMax` | How many can be stacked on top of each other |
| `UnitsPerCase` | How many units come in one ordered case |
| `DeliveryDelayHours` | Hours between ordering and the box arriving |
| `DeliversNextDay` | If ticked, the box arrives at opening time the next day instead, whatever the hour |
| `RequiresRefrigeration` | Declared for you to read. The fridge already restricts by category |

`DeliversNextDay` wins over `DeliveryDelayHours` when both are set.

---

## Getting the size right

`FootprintCm` and `StackHeightCm` decide how many units fit on a shelf and how many fit in a cardboard box. Measure the mesh rather than guessing:

1. Open the static mesh.
2. Read the **Bounds** in the details panel.
3. `FootprintCm.X` is the width, `FootprintCm.Y` is the depth, `StackHeightCm` is the height.

Add a millimetre or two for a visible gap between units on the shelf. Take one away and they interpenetrate.

What follows from the numbers:

- A wide product gives fewer facings per row.
- A tall product is refused by rows with low clearance and ends up on a top shelf.
- A large product gives fewer units per case, since the box capacity comes from the same numbers.

---

## Pricing

`CostPrice` is what you pay per unit, `SuggestedPrice` is what you sell it for. A case costs `CostPrice` times `UnitsPerCase`, times the market quote of the day. See [Rent, tax and the wholesale market](../store/economy.md).

---

## What you get for free

Once the asset exists and it is in `DA_App_Market.AvailableProducts`:

- It appears in the Market app, with its icon, price and case size.
- Ordering it spawns a cardboard box sized to fit it, with the right number of units inside and its icon on the docket.
- It goes on any shelf whose type accepts its category.
- Customers whose archetype prefers that category come looking for it.
- It scans at the checkout, groups on the receipt and shows up in the daily report.

None of that needs a line of Blueprint.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
