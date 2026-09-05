# How a customer thinks

A customer is a Character with a brain component and a very small behaviour tree. Almost all of the decision making is in the brain, and the tree does two things: move, and act when you arrive.

```
BT_Customer
└── Sequence   (service: BTS_CustomerBrain)
    ├── Move To  (TargetLocation)
    └── BTT_ActAtTarget
```

The service calls `BP_CustomerBrain.UpdateTarget()` and writes the result into the blackboard. The task calls `ActAtTarget()`. Neither of them decides anything.

**`UpdateTarget` is the only thing in the project that writes a customer's destination or what they are looking at.** No task touches either. That rule exists because the alternative was tried in another template: seven tasks setting a focus and three clearing it, and characters staring at things forever.

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
| `Browsing` | Standing in front of a shelf, looking at it, waiting out `BrowseSeconds` |
| `Picking` | Taking a unit off the shelf into the basket |
| `Queuing` | In line at a till |
| `WaitingForCheckout` | At the post, unloading and waiting for the scanning to finish |
| `PayingCash` / `PayingCard` | Holding out a note or a card |
| `Leaving` | Walking to the exit |
| `LeavingAngry` | The same walk, with the rating docked |

---

## Choosing a shelf

A customer only walks to a shelf that has something they want. That sounds obvious and it was not the first implementation: before it, an empty store had customers wandering from shelf to shelf forever.

`ChooseShelf` looks at every shelf, asks what it is actually holding, and keeps the ones carrying a product this archetype wants and can afford. A shelf that fails is struck off for this customer's visit.

When the list comes up empty, the customer does not loop. They report a stock-out, which dents the store rating, and then either go and pay for whatever is already in their basket or walk out.

The purchase rule lives on the **archetype**, not on the brain:

```
DA_Customer_*.Wants(Product, Budget) → bool
```

Preferred category, and price under the remaining budget. Overriding it in your own archetype child is where you would put "never buys frozen food after 6pm" or anything else.

---

## Standing in front of a shelf

Customers do not all walk to the same point. `ApproachPointFor` picks a random spot on a ring at `StandOffCm` from the shelf and traces from eye height to check the shelf is actually visible from there. It tries `ApproachTries` times and gives up, striking the shelf off, rather than walking into a wall.

That is what spreads four customers around a round display instead of stacking them on one side, and it is also what stops a shelf pushed against a wall from sending customers outside the building.

Two known limits, said plainly:

- A partition **lower than eye height** does not block the trace, so a point behind a low wall can be accepted.
- Nothing reserves the point. Two customers can pick spots next to each other.

---

## Patience

`PatienceSeconds` on the archetype only drains **while queuing**, and it stops for the customer currently being served. Taking a long time to scan a big basket does not make the person you are serving angry, which is the right way round.

When it runs out, the customer leaves the queue cleanly, goes to `LeavingAngry`, and the store rating drops.

---

## Where they look

The service reasserts orientation every tick, with one writer and two regimes:

- **Moving**: the character rotates toward movement, the engine's own `bOrientRotationToMovement`.
- **Stopped with something to look at**: the AI controller's focus points them at it.

They only look at something once they have stopped. Walking toward a queue slot while already staring at it made them approach sideways.

While browsing, the thing they look at is the shelf, and the focus is cleared when browsing ends.

---

## Speech bubbles

Customers and employees say things over their heads at specific moments. It is a Data Asset system, so a new line is content and not code.

| Piece | What it is |
|---|---|
| `E_ReactionEvent` | The catalogue of moments: `CustomerNotFound`, `CustomerImpatient`, `EmployeeIdle` |
| `BP_ReactionDataAsset` | One asset per moment: the event, a list of lines, and how long the bubble stays |
| `BP_ReactionComponent` | A screen-space widget component 110 cm above the capsule centre, on both the customer and the employee |
| `DT_ReactionTextStyles` | The rich text styles: `Default`, and `Accent` for the highlighted word |

A line is a `Text` with two pieces of markup: an `Accent` rich text tag around the word to emphasise, and `{Subject}` where the caller's subject goes. A customer who cannot find anything says `Couldn't find any {Subject}…` with their preferred category in amber.

Adding a moment is an enum entry, a `DA_Reaction_*`, and one call to `React` from wherever it happens.

---

## What a customer carries

A customer's basket holds real stock that has already left your shelves. That is why customers are written into the save file: losing them mid-visit would quietly destroy inventory you paid for.

An in-progress sale is not saved. It is cancelled and the goods are returned, which is exact rather than approximate, because money only moves in `FinishSale` and that call is all or nothing.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
