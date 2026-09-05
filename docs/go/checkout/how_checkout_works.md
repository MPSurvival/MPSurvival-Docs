# How the checkout works

Serving a customer is a sequence of gestures, not a screen you click through. They queue, they unload onto the belt, you pick up the scanner and scan each item, then they pay. Money is on the [next page](payment_and_change.md).

---

## A till needs a scanner assigned to it

A checkout takes customers only when `Open` is ticked **and** `PairedScanner` points at a scanner. If the only checkout in the store fails that test, **no customer is born at all**, so a badly assembled till shows itself immediately instead of producing shoppers who queue and leave angry.

| Field | What it does | Shipped |
|---|---|---|
| `Open` | Whether the till is open | |
| `PairedScanner` | The `BP_BarcodeScanner` that belongs to this till | |
| `QueueCapacity` | How many customers can queue | `6` |
| `AcceptedPayments` | Which of `Cash` and `Card` a customer may offer | both |
| `BillValues` | The notes customers pay with | `1 · 5 · 10 · 20` |
| `AttendedRadiusCm` | How close someone must be for the till to count as attended | `400` |

A closed till switches its screen off, so the world says so too.

---

## The queue is a spline you draw

`BP_QueueComponent` is a spline on the counter. **Point 0 sits at the till and the queue runs backwards from there**: point 0 is the customer being served, point 1 is the person behind them. Bend it around a gondola or into an L if your floor plan needs it — everyone faces the next point along, so a curved queue looks right with nothing to aim by hand.

| Field | Where |
|---|---|
| `SlotSpacingCm` | On the queue component: the gap between two customers |
| `QueueCapacity` | On the checkout: how many fit |

Two components on the counter finish the job:

| Component | What it is |
|---|---|
| `BuyerSpot` | Where the customer being served stands. They walk there |
| `CashierSpot` | What they look at, and where a hired cashier stands |

Move either one in the viewport and the behaviour follows. The same setup works whether you are serving or an employee is.

---

## The belt

The customer puts their shopping on the `DropZone`, and each unit becomes a real actor on the belt. How many fit is decided by the size of that box component: anything that does not fit stays in the basket and slides down as soon as a place frees up.

| Field | What it does | Shipped |
|---|---|---|
| `ItemSpacingCm` | Gap between two items along the belt | `11` |
| `BeltSpeedCmS` | How fast items slide forward | `25` |
| `ScatterCm` | Random sideways offset when an item is put down | `6` |
| `ScatterYawDeg` | Random yaw when an item is put down | `12` |

`ItemSpacingCm` is `11` because that is the widest product in the shipped catalogue. Lower and items overlap, higher and you waste belt.

---

## Scanning

You scan with the scanner **that belongs to this till**, held in your hands. Without it, an item on the belt has no outline and no prompt: pick up the scanner, scan, put it back. Two tills side by side do not lend each other their scanners.

A scanned item flies into the bag, and the beep comes from the scanner in your hands. The bag only shows itself while a customer is at the post.

---

## The receipt and the screen

Scanning the same product twice puts `×2` on one line rather than adding a second line, whatever order things were scanned in. The customer screen shows one row per line — thumbnail, name, quantity, unit price, line total — and scrolls when the basket is big. With nobody at the till it reads `NO CUSTOMER`.

Finishing a sale does not clear the receipt: the last ticket stays up until the next customer steps forward, like a real till.

---

## The cord

The scanner is tied to its till by a coiled cable, 120 cm long, so it cannot be lost or swapped with the one next door. Walk past the leash while holding it and the scanner is pulled out of your hands rather than slowing you down.

Its home is the `ScannerCradle` component on the counter. Move that in the viewport and move the cord's anchor with it.

Two things the cord does not do: it does not wrap around obstacles, so a tight cable passes through the counter, and its coils are drawn by the material rather than modelled.

---

## An unattended till

When a customer reaches a till with nobody within `AttendedRadiusCm`, a notification appears top left, once per arrival.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
