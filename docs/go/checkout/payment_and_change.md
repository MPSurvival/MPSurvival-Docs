# Cash, change and the card terminal

There is no payment screen. You take what the customer holds out, and you finish the sale by hand.

The rule underneath all of it: **what you do is what gets recorded.** Miscount the change and you lose the difference. Type the wrong amount into the terminal and that is what you charge.

---

## What the customer holds out

When the last item is scanned, the customer holds out a real object — a card or a banknote — in their right hand. You click the object, not the person: customers are not interactable, so the outline lands on the card and never on a shopper's chest.

Which one they offer comes from `AcceptedPayments` on the till. Set it to `Cash` only and you count out change on every single sale.

---

## Cash

1. The customer holds out the smallest note from `BillValues` that covers the total: `$6.40` gets a `$10`.
2. Press `E`. The note goes into the till and the drawer slides open.
3. Click the money piles in the drawer, one at a time, to build the change.
4. Push the drawer shut to finish the sale.

The drawer is open exactly when you owe change.

**Each pile of money is its own actor.** Four bundles of notes and four trays of coins, each carrying its own denomination: aiming at the `$20` bundle prompts `$20`, and clicking it adds `$20`.

**To change what a drawer holds**, move those `BP_DrawerMoney` child actors around in the Details panel and set their denomination. A fifth denomination also needs a mesh and a row in the money texture atlas.

---

## The panel on the drawer

While the drawer is open, a readout sits on it:

```
RECEIVED  −  TOTAL   =   CHANGE DUE
                GIVEN
```

`GIVEN` is what you have put on the counter so far, red while it does not add up and green when it does, with an arrow saying whether to add more or take some back. It does not show you the remainder — the subtraction is yours to do.

Give too much back and you are down by the difference. Give too little and you keep it.

---

## Card

The terminal is mounted on the counter **facing the cashier**, not the queue.

1. The customer holds out a card. Click it: the card flies to the terminal, which wakes up.
2. Press `E` to lean in. The keypad appears.
3. **Type the amount** on the keypad.
4. Green approves, red closes without charging.

What you type is what is charged. You can back out at any point before green, and the customer stays at the front of the queue until the sale actually completes. They do not enter a PIN.

---

## Where to change things

| What | Where |
|---|---|
| Which notes customers pay with | `BillValues` on `BP_CheckoutBase` |
| Cash only, card only, or both | `AcceptedPayments` on `BP_CheckoutBase` |
| What is in the drawer | The `BP_DrawerMoney` child actors on `BP_DrawerBase` |
| How far and how fast the drawer slides | `DrawerTravelCm` (`34`) and `DrawerSpeedCmS` (`60`) |
| The sale sound | `SaleSound`, played from the `SaleSoundSpot` component on the counter |

`SaleSoundSpot` is a plain scene component 10 cm above the counter top, and you can drag it. A sound played at the actor's origin would play from inside the furniture.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
