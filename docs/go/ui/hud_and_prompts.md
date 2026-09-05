# The HUD and the interaction prompt

The interface is in four layers, and they never mix.

| Layer | Where | What belongs there |
|---|---|---|
| Permanent HUD | Top right | Your money, the day, the clock, the store rating |
| Player context | Top left, and around the crosshair | Notifications, the interaction label, the keys you can press right now |
| Diegetic | In the world | Anything belonging to an object: the till display, the card terminal, the office computer, the box docket |
| Applications | Full screen | The build menu, the daily report, the menus |

When you add a widget, the question that places it is **whose information is this**. The till display shows the till's receipt, so it lives on the till. The carry panel shows your keys, so it belongs to you even though it is drawn next to the box.

---

## The status panel

| Element | Detail |
|---|---|
| Balance | Thousands separator, two decimals, full opacity |
| Currency symbol | A **separate** text block, in `MoneyGreen` |
| Day line | `DAY 1 · MON`, at 0.55 opacity |
| Clock | Right-aligned on the balance's width |
| Rating ring | Fill is rating ÷ 5, value in the middle |

The weekday names are `WeekdayNames` on the HUD, in `Settings`. The panel is a `Border` using a native `RoundedBox`, radius 6, with no texture behind it, so reskinning it is the Details panel and not an asset.

**To restyle the rating ring**, open `MI_RatingRing`:

| Parameter | What it does |
|---|---|
| `RingRadius` | Size |
| `RingThickness` | Stroke width |
| `EdgeFeather` | Softness of the edge |
| `TrackOpacity` | The unfilled part |

Its colour ramp is inside the material: red at 0, orange around 1.7, yellow around 3.3, green at 5.

---

## Notifications

They stack at the top left and fade out after `NotificationSeconds`. Two things post them today: a customer arriving at a checkout with nobody within `AttendedRadiusCm`, and the store rating crossing a whole number.

**To post one of your own**, call the store manager's notification dispatcher.

---

## The crosshair and the prompt

`BP_InteractionComponent` on the character decides what you are looking at and drives the crosshair. Settings, in `Settings|Interaction`:

| Setting | What it does |
|---|---|
| `TraceLength` | How far you can reach. Also how far `G` places an object |
| `FocusBlendSpeed` | How fast the crosshair reacts |
| `PromptRisePx` | How far the label rises when a target is acquired |

The label is quiet: D-DIN at 16, 0.55 opacity, no outline. It says what the thing is, not which key to press, since the key is `E` everywhere.

The crosshair is the whole prompt: no key badge, no hold ring. **Hold-to-interact works** through `HoldSeconds` on the interactable, and when you use it, the fill belongs in `M_Crosshair`.

!!! warning
    If `CrosshairClass` is empty on the interaction component, nothing ever gets a prompt. It is the first place to look if interaction stops entirely.

---

## Making something interactive

Add a `BP_InteractableComponent` and wire its `OnInteracted` event in the actor's own graph. There is no interface to implement and no parent class to inherit.

| Field | What it does |
|---|---|
| `Label` | The text under the crosshair |
| `HoldSeconds` | `0` for a press, more for a hold |
| `Enabled` | Whether it responds at all |

`Enabled` also controls the outline, so an interactable switched off at runtime stops looking live in the same frame.

The component broadcasts `OnFocusChanged` when the aimed target changes, which is how a carried box knows what you are pointing at without looking for the player.

---

## The action panel

While you are carrying something, a panel of key prompts sits beside it. See [Carrying](../delivery/carrying.md).

It also carries a hint line: when edit mode refuses to pick something up, the reason is printed under the keys in the `Danger` colour.

---

## Numbers and locale

Numbers are formatted in the machine's culture. On a French machine the balance reads `$1 000,00` and the rating `3,0`; on an English one, `$1,000.00` and `3.0`. Worth knowing when you take screenshots.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
