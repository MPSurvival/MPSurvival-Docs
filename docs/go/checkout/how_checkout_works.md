# How the checkout works

Serving a customer is a sequence of physical gestures, not a screen you click through. They queue, they unload onto the belt, you pick up the scanner and scan each item, they hand you money or a card, and you finish the sale by hand.

This page covers the queue, the belt and the scanning. Money is on the [next page](payment_and_change.md).

---

## A checkout is only usable if it has a scanner

```
CanServe()  =  Open  AND  PairedScanner is set
```

That one function is the whole definition, and it has two readers: a customer asking whether it can join the queue, and the spawner asking whether the store is worth walking into at all.

The consequence is worth spelling out. A checkout with no scanner assigned takes no customers, and if it is the only checkout in the store, **no customer is born**. A shop with no way to sell is treated exactly like a shop with nothing to sell.

That is deliberate. A badly assembled checkout shows itself immediately, instead of producing a stream of customers who queue and leave angry.

A closed checkout also switches its screen off, so there is something in the world that says so.

| Field | What it does | Shipped default |
|---|---|---|
| `Open` | Whether the till is open. Written only by `SetOpen` | |
| `PairedScanner` | The `BP_BarcodeScanner` that belongs to this till | |
| `QueueCapacity` | How many customers can queue | `6` |
| `AcceptedPayments` | Which of `Cash` and `Card` a customer may offer | both |
| `BillValues` | The notes customers pay with | `1 · 5 · 10 · 20` |
| `AttendedRadiusCm` | How close someone must be for the till to count as attended | `400` |

---

## The queue is a spline you draw

`BP_QueueComponent` is a child of `SplineComponent`, and it knows nothing about shops. It handles actors and a curve.

**Point 0 sits at the till, and the queue runs backwards from there.** Point 0 is where the customer being served stands, point 1 is the person behind them, and so on. Draw it around a gondola or bend it into an L if your floor plan needs that.

| Field | Where it lives |
|---|---|
| `SlotSpacingCm` | On the queue component: the gap between two customers |
| `QueueCapacity` | On the checkout: how many fit |

Capacity has one owner, and it is the checkout. The queue component's own capacity is written by `ConfigureQueue` at `BeginPlay` and never edited by hand.

Everyone in the queue faces the next point along, so a curved queue looks right without anything being aimed by hand.

Two components on the counter finish the job:

| Component | What it is |
|---|---|
| `BuyerSpot` | Where the customer being served stands. They walk there, they are not teleported |
| `CashierSpot` | What they look at, and where a hired cashier stands |

Move either one in the viewport and the behaviour follows. There is no reference to a cashier actor anywhere, which is why the same setup works whether you are serving or an employee is.

---

## The belt

The customer puts their shopping down on the `DropZone`, a box component sitting over the drop plate. Each unit becomes a `BP_CheckoutItem`, a real actor carrying its own product, its own mesh and its own interactable.

There is no parallel list. What is on the belt is what exists on the belt.

| Field | What it does | Shipped default |
|---|---|---|
| `ItemSpacingCm` | Gap between two items along the belt | `11` |
| `BeltSpeedCmS` | How fast items slide forward | `25` |
| `ScatterCm` | Random sideways offset when an item is put down | `6` |
| `ScatterYawDeg` | Random yaw when an item is put down | `12` |

`ItemSpacingCm` is 11 because that is the widest footprint in the shipped product catalogue. Go lower and items overlap; go higher and you waste belt.

How many items fit is decided by the size of the `DropZone` box. Anything that does not fit stays in the customer's basket, and a new item slides down as soon as a place frees up.

---

## Scanning

You can only scan with the **scanner that belongs to this till**, held in your hands. Without it an item on the belt shows no outline and no prompt at all. It is not an interaction that refuses, it is an interaction that does not exist.

The gesture becomes: pick up the scanner, scan, put it back. That is what stops the scanner being a prop with a cord attached.

Two tills side by side do not lend each other their scanners. The test is `CarriedActor == Checkout.PairedScanner`.

The item decides its own state. It subscribes to the carry component's `OnCarryChanged` and enables or disables its own interactable, which is why the outline appears the moment you pick the scanner up and not a frame later.

A scanned item flies into the `Bag` component on a short arc, and the beep comes from the scanner in your hands rather than from the counter.

The bag only shows itself while a customer is at the post. A till standing empty with an open carrier bag on it means nothing.

---

## The receipt and the screen

Scanning adds a line to the receipt, and scanning the same product again finds its existing line and increases the count. Three cartons of milk are one line reading `×3`, not three lines, whatever order they were scanned in.

`BP_CheckoutScreen` is a child of `BP_ScreenBase` with two values set in its Details panel and no graph at all. It shows one row per receipt line: thumbnail, name, quantity, unit price, line total. The rows scroll, so a big basket does not fall off the display.

When no customer is at the till, the table collapses and `NO CUSTOMER` sits across the screen on the diagonal.

The screen subscribes to `OnReceiptChanged` on the checkout, which fires at the end of scanning an item, at the end of accepting a basket, and at the end of a sale.

A sale does **not** clear the receipt. `PresentBasket` does, when the next customer steps up. The last ticket stays on screen until then, like a real till.

---

## The cord

The scanner is tied to its till by a coiled cable, so it cannot be lost or mixed up with the one next door.

`CableMaxLength` is 120 cm. Walk past the leash while holding the scanner and it is pulled out of your hands, rather than slowing you down, because taking control of the player's movement away is worse than dropping a tool.

The scanner's home is `ScannerCradle`, a component on the counter. Move it in the viewport and the cord's anchor should move with it.

Two things the cable cannot do, and neither has a setting: it does not wrap around obstacles, so a tight cord will pass through the counter, and its silhouette is a smooth tube rather than a real helix. The coils are drawn by the material.

---

## An unattended till

When a customer arrives at a till with nobody within `AttendedRadiusCm` (400 cm by default), a notification appears at the top left of the screen. It fires on arrival rather than every tick, so it says something once and stops.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
