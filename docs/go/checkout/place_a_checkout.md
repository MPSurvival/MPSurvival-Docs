# Put a checkout in your level

A till is one actor, `BP_Checkout`, plus a scanner it has to know about. Forgetting the second half is the most common reason a store produces no customers at all.

---

## The steps

1. Drag `BP_Checkout` into the level and place the counter.
2. Drag in a `BP_BarcodeScanner`, or use the one that comes with the counter, and drop it in the cradle.
3. Select the checkout and set **`PairedScanner`** to that scanner.
4. Check that `Open` is ticked.
5. Select the `BP_QueueComponent` and drag its spline points to draw the queue. **Point 0 stays at the counter**, the rest run backwards.
6. Move `BuyerSpot` and `CashierSpot` if your counter is not the shipped one.
7. Make sure the nav mesh covers the whole queue, and that the queue does not run through a shelf.

Then walk it once in game. A queue that reads well is one whose last point you can reach without squeezing past a gondola.

---

## When something is wrong

| Symptom | Cause |
|---|---|
| No customers spawn at all | Every checkout is closed, or has no `PairedScanner` |
| Customers walk in and leave angry immediately | Same, or nothing on the shelves they want |
| Items cannot be scanned | You are not holding the scanner of that till |
| Two customers stand in the same place | The queue spline is shorter than `SlotSpacingCm × QueueCapacity` |
| The customer being served stands off to the side | `BuyerSpot` has drifted on that instance |
| The cord hangs oddly or points at nothing | The cable's anchor is not lined up with the scanner's cord exit |

---

## A till placed before a change to the Blueprint

An actor **already placed** in a level does not gain components or variables added to its Blueprint afterwards. Delete the placed actor and place a fresh one.

What comes back with it is everything that lives on the class, including the shape of the queue spline. What does not is anything you set on that instance — `PairedScanner` and `SaleSound` in particular. Note them down before you delete.

---

## Two or more tills

Each one has its own queue, its own scanner and its own drawer, and a customer picks a checkout when it decides to pay.

- Each till needs its **own** scanner. They do not share.
- The counter mesh sticks out about 23 cm past its placement footprint on one side, so two tills placed edge to edge in build mode overlap slightly without being refused. Leave a cell between them.

---

## Opening and closing

`SetOpen(NewOpen)` on the checkout opens or closes a till and refreshes its screen. An employee taking the `ManCheckout` task is what calls it today; trading hours, a power cut or a till that closes when nobody is around are one call to the same function.

A closed till keeps whoever is already in its queue and stops accepting anyone new.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
