# The HUD and the interaction prompt

The interface is organised into four layers, and they never mix.

| Layer | Where | What belongs there |
|---|---|---|
| Permanent HUD | Top right | Your money, the day, the clock, the store rating |
| Player context | Top left, and around the crosshair | Notifications, the interaction label, the keys you can press right now |
| Diegetic | In the world | Anything belonging to an object: the till display, the card terminal, the office computer, the box docket |
| Applications | Full screen | The build menu, the daily report, the menus |

The question that decides where a widget goes is **whose information is this**. The till display shows the till's receipt, so it lives on the till. The carry panel shows your available keys, so it belongs to you even though it is drawn next to the box.

---

## The status panel

One block, top right, on a grid. The rating ring on the left, the balance in large type, and the day and clock on a secondary line.

```
       DAY 1 · MON        8:00 AM
  ◯    $1,240.00
```

| Element | Detail |
|---|---|
| Balance | Thousands separator, two decimals, full opacity |
| Currency symbol | A **separate** text block, in `MoneyGreen` |
| Day line | `DAY 1 · MON`, at 0.55 opacity |
| Clock | Right-aligned on the balance's width, no separator in the middle |
| Rating ring | Fill is rating ÷ 5, value in the middle |

The weekday names are `WeekdayNames` on the HUD, in `Settings`. The day number maps to a weekday through one function on the store manager, so payroll, shift toggles and the HUD all agree on what day it is.

The panel is a `Border` using a native `RoundedBox`, radius 6, with no texture behind it. Reskinning it is the Details panel, not an asset.

Money and rating arrive through the store manager's dispatchers. Only the clock is on a tick, and the text is rebuilt only when the minute changes.

---

## The rating ring

`MI_RatingRing` draws the arc and its colour from one input.

| Parameter | What it does |
|---|---|
| `RingRadius` | Size |
| `RingThickness` | Stroke width |
| `EdgeFeather` | Softness of the edge |
| `TrackOpacity` | The unfilled part |

The colour ramp is inside the material: red at 0, orange around 1.7, yellow around 3.3, green at 5, all desaturated. Because one input drives both the arc and the colour, they cannot disagree, and changing the mood of the rating is four colour swatches.

---

## Notifications

They stack at the top left and fade out after `NotificationSeconds`.

Two things post them today:

- A customer arriving at a checkout with nobody within `AttendedRadiusCm`.
- The store rating crossing a whole number, in either direction.

Posting one from your own code is a call to the manager's notification dispatcher. The HUD subscribes to it exactly like it subscribes to money and rating.

---

## The crosshair and the prompt

`BP_InteractionComponent` is the only thing that decides what you are looking at, and the only thing that drives the crosshair.

It sphere traces from the camera every frame, up to `TraceLength` (220 cm), and the result feeds two things: the crosshair material's `Focus` parameter, which is interpolated rather than snapped, and a single line of text underneath.

| Setting | What it does |
|---|---|
| `TraceLength` | How far you can reach. Also how far `G` will place an object |
| `FocusBlendSpeed` | How fast the crosshair reacts |
| `PromptRisePx` | How far the label rises when a target is acquired |

The label is deliberately quiet: D-DIN at 16, 0.55 opacity, no outline. It says what the thing is, not which key to press. The key is `E` everywhere and does not need announcing.

There is no key badge and no hold ring. Both were built and removed. **Hold-to-interact still works** in code, through `HoldSeconds` on the interactable, it just has no visual feedback, and nothing in the shipped project uses it. If you need it, the fill belongs in `M_Crosshair` rather than on a new badge.

**If `CrosshairClass` is empty on the interaction component, nothing ever gets a prompt.** The component refuses to start ticking rather than half working. It is the first place to look if interaction stops entirely.

---

## Making something interactive

Add a `BP_InteractableComponent` and wire its `OnInteracted` event in the actor's own graph.

There is no interface to implement and no parent class to inherit.

| Field | What it does |
|---|---|
| `Label` | The text under the crosshair |
| `HoldSeconds` | 0 for a press, more for a hold |
| `Enabled` | Whether it responds at all |

`Enabled` also controls the outline. An interactable switched off at runtime loses its outline in the same frame, which matters because a disabled interactable still blocks the trace and would otherwise look live.

The component broadcasts `OnFocusChanged` when the aimed target changes, at the edge rather than every tick. That is how a carried box knows what you are pointing at without ever looking for the player.

---

## The action panel

While you are carrying something, a panel of key prompts sits beside it. See [Carrying](../delivery/carrying.md).

It also carries a hint line: when edit mode refuses to pick something up, the reason is printed under the keys in the `Danger` colour.

---

## Numbers and locale

Numbers are formatted in the machine's culture, because `To Text (Float)` goes through the engine's number formatter.

On a French machine the balance reads `$1 000,00` and the rating `3,0`; on an English one, `$1,000.00` and `3.0`. That is the right behaviour for a shipped game. It is worth knowing if you are taking screenshots, since a dollar sign next to a decimal comma reads as a bug.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
