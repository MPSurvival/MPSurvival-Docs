# Colours, fonts and reskinning

Colours live in the Details panel of the widget that draws them, and this page is the table they all come from.

---

## The palette

Values are **linear**, which is what the Unreal colour picker expects. The hex is the sRGB equivalent, for reference.

| Token | Linear | Used for |
|---|---|---|
| `Ink` | `0.92, 0.94, 0.96` (`#F7F9FB`) | All text. Full opacity for data, **0.55** for labels |
| `PanelSlate` | `0.006, 0.010, 0.014` at **0.78** | Panel backgrounds, as a native `RoundedBox`, radius 6, edge at 0.08 |
| `MoneyGreen` | `0.019, 0.250, 0.078` (`#26894F`) | The currency symbol, and nothing else |
| `RatingPoor` | `0.553, 0.093, 0.078` (`#C4564F`) | Rating 0 / 5 |
| `RatingFair` | `0.577, 0.258, 0.068` (`#C88B4A`) | Rating ~1.7 / 5 |
| `RatingGood` | `0.584, 0.515, 0.105` (`#C9BE5B`) | Rating ~3.3 / 5 |
| `RatingGreat` | `0.159, 0.392, 0.112` (`#6FA85E`) | Rating 5 / 5 |
| `AppMarket` | `0.055, 0.180, 0.320` (`#427699`) | Market title bar |
| `AppStructures` | `0.319, 0.147, 0.054` (`#996B42`) | Structures title bar |
| `AppHiring` | `0.114, 0.055, 0.320` (`#5F4299`) | Hiring title bar |
| `AppStaff` | `0.319, 0.055, 0.153` (`#99426D`) | Staff title bar |
| `AppAds` | `0.320, 0.320, 0.054` (`#999942`) | Ads title bar |

---

## The currency rule

**Every currency symbol in the game is `MoneyGreen`**: the HUD, shelf price labels, the till, the card terminal, every app, the daily report.

In practice the symbol is its own `TextBlock` next to the number, not part of the same string. One extra widget per price, and it is the most recognisable thing about the interface.

The interaction prompt is the one place it is not applied, because that line is a single text block.

---

## Picking a colour for a new app

All five app accents sit at **S 0.57, V 0.60** in HSV. Only the hue moves, and the hue says what the app is for:

| Camp | Hues | Meaning |
|---|---|---|
| Cold | Market 204, Hiring 260 | Catalogues you buy from |
| Warm | Structures 28, Staff 330, Ads 60 | Things you own and manage |

Pick your camp, then take the widest hue gap left in it. **Stay out of the green band**: `MoneyGreen` is at 148, and money is on nearly every screen.

The title bar is the **only** accent. The rest of the chrome stays on `Ink` and `PanelSlate`, so six apps read as one machine.

---

## Type

D-DIN, and nothing else, in `Textures/Fonts/`.

| Use | Size |
|---|---|
| Screen title | 56 Bold |
| Column headers | small, at 0.55 opacity |
| Body | 18 to 22 Regular |
| Interaction label | 16, 0.55 opacity, no outline, slight letter spacing |

**A screen in the world halves every detail.** A 2 px line on a monitor across the room is invisible, and type that looks right in the Designer is usually too light in game. Judge both at the distance you actually read them from.

---

## Rules that apply everywhere

**No tooltips.** There are none in the project, and there should be none in yours. A tooltip needs a hover and a wait, which does not exist in a first person game, and it hides information the screen should have shown. If a piece of text is worth reading, give it a column.

**Colours are never built in a graph.** A colour is a swatch in the Details panel, or an Instance Editable `LinearColor` variable in a `Style` category when it has to be adjustable. Opacities and line weights follow the same rule.

**Two screens using the same control take the same colour from the table above**, rather than from the widget next door.

---

## Reskinning

1. `BP_HudWidget` for the status panel and its border.
2. `MI_RatingRing` for the four rating colours.
3. `BP_ComputerWidget` for the desktop chrome.
4. The `TitleColor` field on each `DA_App_*` for the accents.
5. `BP_ScreenBaseWidget` and the two till widgets for the world screens.
6. The menu widgets for the front end.

The fonts are two assets. Replace D-DIN with your own face in the font asset and every widget follows.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
