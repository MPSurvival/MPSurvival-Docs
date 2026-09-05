# Put a checkout in your level

A till is one actor, `BP_Checkout`, plus a scanner it has to know about. Getting the second half wrong is the most common reason a store produces no customers at all.

---

## The steps

1. Drag `BP_Checkout` into the level and place the counter where you want it.
2. Drag a `BP_BarcodeScanner` in, or use the one that comes with the counter, and drop it in the cradle.
3. Select the checkout and set **`PairedScanner`** to that scanner.
4. Check that `Open` is ticked.
5. Select the `BP_QueueComponent` and drag its spline points to draw the queue. **Point 0 stays at the counter**, the rest run backwards.
6. Move `BuyerSpot` and `CashierSpot` if your counter is not the shipped one.
7. Make sure the nav mesh covers the whole queue, and that the queue does not run through a shelf.

Then walk it in game once. A queue that reads well is a queue whose last point you can reach without squeezing past a gondola.

---

## The checks that catch most problems

| Symptom | Cause |
|---|---|
| No customers spawn at all | Every checkout is closed, or has no `PairedScanner` |
| Customers walk in and leave angry immediately | Same, or nothing on the shelves they want |
| Items cannot be scanned | You are not holding the scanner of that till |
| Two customers stand in the same place | The queue spline is shorter than `SlotSpacingCm × QueueCapacity` |
| The customer being served stands off to the side | `BuyerSpot` has drifted, or the actor has stale instance overrides |
| The cord hangs oddly or points at nothing | The cable's anchor is not lined up with the scanner's cord exit |

The queue length is worth a second look. Distance along the spline is clamped to the curve's length, so a short curve piles everyone at the far end rather than throwing them into the void, and it does not look broken so much as wrong.

---

## Instance overrides

This one is not specific to the checkout, but the checkout is where it bites hardest.

When a component or a variable is added to a Blueprint, actors **already placed** in a level do not get it. They keep the engine defaults, or empty values, and writing the property on the class does not reach them.

The fix is to delete the placed actor and place a fresh one. Everything that lives on the class comes back, including the queue spline, whose shape lives on the component template rather than on the instance.

What does **not** come back is anything you set on that instance: `PairedScanner` and `SaleSound` in particular. Note them down before you delete.

---

## Two or more tills

Nothing stops you. Each one has its own queue, its own scanner and its own drawer, and a customer picks a checkout when it decides to pay.

Two things to be aware of:

- Each till needs its **own** scanner. They do not share.
- The counter mesh sticks out about 23 cm past its placement footprint on one side, so two tills placed edge to edge in build mode will overlap slightly without being refused. Leave a cell between them.

---

## Opening and closing

`SetOpen(NewOpen)` is the only way `Open` changes, and it refreshes the screen at the same time, so there is no way to end up with a closed till showing a live display.

Nothing calls it automatically today except an employee taking the `ManCheckout` task. Trading hours, a power cut, or a till that closes when nobody is around are all one call to that function.

A closed till keeps whoever is already in its queue. It just stops accepting anyone new.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
