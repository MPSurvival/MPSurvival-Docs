# Cash, change and the card terminal

Taking payment is a set of gestures on objects in the world. There is no payment screen, and there is no confirmation dialog. You take what the customer holds out, and you finish the sale with your hands.

The rule underneath all of it: **what you actually do is what gets recorded.** Miscount the change and you lose the difference. Type the wrong amount into the terminal and that is what you charge.

---

## What the customer holds out

When the last item is scanned, `BeginPayment` picks a payment type from the till's `AcceptedPayments` and the customer holds something out. Not their hand: a real object, `BP_PaymentOffer`, sitting in a child actor component in their right hand.

It is one actor that adapts, with two verbs, `OfferCard()` and `OfferCash()`. It shows a card or a banknote, turns its collision and its interactable on, and waits.

You click the object, not the person. Customers are not interactable at all, which is why the outline lands on the card and not on a shopper's chest.

A till set to `Cash` only means you count out change on every single sale. That is its whole effect, and it is enough.

---

## Cash

1. The customer holds out a note. `CashOffer()` is the smallest note from `BillValues` that covers the total: `$6.40` gets a `$10`, `$63` gets two `$50`s worth.
2. Press `E`. The note leaves their hand, goes into the till, and the drawer slides open.
3. Click the money piles in the drawer, one at a time, to build the change.
4. Push the drawer shut to finish the sale.

The drawer is open exactly when you owe change. There is no separate "drawer open" state, because two names for one truth is a desynchronisation waiting to happen.

**Each pile of money is its own actor.** Four bundles of notes and four trays of coins, each a `BP_DrawerMoney` carrying its own denomination and its own label. Aiming at the `$20` bundle prompts `$20` and clicking it adds `$20`. The prompt and the amount are two fields on the same actor, so they cannot disagree.

Changing what a drawer holds is moving those child actors around in the Details panel and setting their denomination. Adding a fifth denomination needs a mesh and a row in the money texture atlas as well.

---

## The panel on the drawer

While the drawer is open, a small readout sits on it:

```
RECEIVED  −  TOTAL   =   CHANGE DUE
                GIVEN
```

`GIVEN` is what you have put on the counter so far. It is red while it does not add up and green when it does, with an arrow telling you whether to add more or take some back.

**It does not show you the remainder.** The subtraction is yours to do, otherwise counting change is just watching a number reach zero.

The panel disappears with the drawer.

---

## What gets recorded

`FinishSale` does not log the receipt total. For a cash sale it logs:

```
Total + ( ChangeDue − ComposedChange )
```

Give too much back and you are down. Give too little and you are up. There is no penalty rule anywhere, because none is needed.

---

## Card

The card terminal is `BP_CardTerminal`, a child of `BP_ScreenBase`, mounted on the counter and **facing the cashier**, not the queue. That is what decides where it sits on the counter.

1. The customer holds out a card. Click it. The card flies to the terminal and the terminal wakes up.
2. Press `E` to lean into the terminal. The keypad appears.
3. **Type the amount** on the keypad.
4. Green approves, red closes without charging.

Typing the amount is the same design as counting the change: what you enter is what is charged. Getting it wrong costs or earns you money, with no extra rule.

The customer does not enter a PIN. That belongs to them.

Because the sale is closed by the terminal rather than by the customer, you can back out at any point before you press green. The customer stays at the front of the queue until the sale actually completes.

The terminal is dark until a card is in it. `PowerOn` shows the display and the card and enables the interactable in one call, `PowerOff` puts it back to sleep.

---

## Where to change things

| What | Where |
|---|---|
| Which notes customers pay with | `BillValues` on `BP_CheckoutBase` |
| Cash only, card only, or both | `AcceptedPayments` on `BP_CheckoutBase` |
| What is in the drawer | The `BP_DrawerMoney` child actors on `BP_DrawerBase` |
| How far and how fast the drawer slides | `DrawerTravelCm` (34) and `DrawerSpeedCmS` (60) |
| The sale sound | `SaleSound`, played from the `SaleSoundSpot` component on the counter |

`SaleSoundSpot` exists because a sound played at an actor's origin plays from inside the furniture, and the occlusion trace then hits the counter before it reaches your ears. It is a plain scene component sitting 10 cm above the counter top, and you can drag it.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
