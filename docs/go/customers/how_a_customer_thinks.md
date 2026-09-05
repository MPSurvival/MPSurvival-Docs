# How a customer thinks

A customer walks in, finds a shelf that has something they want, browses it, takes a unit, repeats until their basket is full, queues, pays and leaves. Everything they decide is in `BP_CustomerBrain`; the behaviour tree only moves them and acts when they arrive.

---

## The states

`E_CustomerState` drives the loop:

```
Entering → Browsing → Picking → Queuing → WaitingForCheckout → PayingCash / PayingCard → Leaving
                                                                                     ↘ LeavingAngry
```

| State | What is happening |
|---|---|
| `Entering` | Walking in from the spawner |
| `Browsing` | Standing in front of a shelf, waiting out `BrowseSeconds` |
| `Picking` | Taking a unit off the shelf into the basket |
| `Queuing` | In line at a till |
| `WaitingForCheckout` | At the post, unloading and waiting for the scanning to finish |
| `PayingCash` / `PayingCard` | Holding out a note or a card |
| `Leaving` | Walking to the exit |
| `LeavingAngry` | The same walk, with the store rating docked |

---

## Choosing a shelf

A customer only walks to a shelf that is actually holding something their archetype wants and can afford. A shelf that fails is struck off for the rest of their visit.

When nothing is left on the list, they report a stock-out — which dents the store rating — and either pay for what is already in their basket or walk out. They never wander in circles.

The buying rule is a function on the archetype, `Wants(Product, Budget)`: preferred category, and price under the remaining budget. Overriding it in your own archetype is how you get "never buys frozen food after 6pm". See [Add a customer archetype](add_an_archetype.md).

---

## Standing in front of a shelf

They pick a random spot on a ring at `StandOffCm` from the shelf and check that the shelf can be seen from there, which spreads four customers around a round display instead of stacking them on one side. A shelf they cannot see from anywhere is struck off rather than walked into.

Two limits worth knowing: a partition **lower than eye height** does not block that check, and nothing reserves a spot, so two customers can end up side by side.

---

## Patience

`PatienceSeconds` on the archetype drains **only while queuing**, and it stops for the customer being served. Taking a long time to scan a big basket does not make the person in front of you angry.

When it runs out they leave the queue, go to `LeavingAngry`, and the store rating drops.

---

## Speech bubbles

Customers and employees say things over their heads at set moments. It is a Data Asset system, so a new line is content and not code.

| Piece | What it is |
|---|---|
| `E_ReactionEvent` | The catalogue of moments: `CustomerNotFound`, `CustomerImpatient`, `EmployeeIdle` |
| `BP_ReactionDataAsset` | One asset per moment: the event, a list of lines, and how long the bubble stays |
| `BP_ReactionComponent` | The bubble itself, on the customer and on the employee |
| `DT_ReactionTextStyles` | The rich text styles: `Default`, and `Accent` for the highlighted word |

A line is a `Text` with two pieces of markup: an `Accent` tag around the word to emphasise, and `{Subject}` where the caller's subject goes. A customer who cannot find anything says `Couldn't find any {Subject}…` with their preferred category in amber.

**To add a moment:** add an entry to `E_ReactionEvent`, create a `DA_Reaction_*`, and call `React` from wherever it happens.

---

## Customers and the save file

Customers are written into the save, basket included, because a basket holds stock that has already left your shelves. A sale in progress is cancelled on load and its goods are returned.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
