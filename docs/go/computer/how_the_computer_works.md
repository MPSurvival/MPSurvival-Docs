# The office computer

Management does not happen in a menu. It happens on a computer sitting on the desk in the back office, and you walk over to it.

Press `E` and the camera leans in, the crosshair and prompt disappear, whatever you were carrying is put down, and the mouse cursor appears. From there it is a desktop: icons, windows with title bars, a taskbar with the time on it. `Tab` or `E` leans you back out.

---

## The desktop

`BP_Computer` is a child of `BP_ScreenBase` and adds almost nothing. It inherits the interactable, the reading pose, the focus and the exit key.

What it does own is `Apps`, an array of `BP_AppDataAsset` in its Details panel. The computer knows nothing about any app: it reads that array, draws an icon for each entry, and instantiates the widget class the asset names.

The order of the array is the order on the desktop. There is no sort field, because nothing would read one.

The chrome is three bands on a 1920 × 1080 canvas:

| Band | Height | Contents |
|---|---|---|
| Title bar | 64 | App icon, app name, your balance, a close button |
| App area | 960 | Whatever the app draws |
| Taskbar | 56 | Time and day |

The title bar takes the app's own colour, and that is the only accent anywhere in the interface. Everything else stays on the two colours the rest of the game uses. See [Colours and fonts](../ui/colours_and_fonts.md).

The close button closes the **app**, not the computer. You leave the computer by stepping back.

---

## The mouse

It is the real mouse cursor, not a drawn one. The controller switches to **Game and UI** input mode with the cursor shown, and a `WidgetInteractionComponent` aimed from the cursor position does the pointing.

Hover, click, scroll and drag are all the engine's own, which is why scroll boxes and buttons behave the way you expect without anything being wired.

Two consequences worth knowing if you build your own screen:

- **Buttons on a world screen have to be non-focusable.** Otherwise `Tab` walks a focus rectangle around them instead of leaving the screen.
- **The widget component's window must not be focusable either**, or it swallows keyboard input.

Both are checkboxes, and both are already set on the shipped screens.

---

## The six apps

| App | What it does |
|---|---|
| Market | Buy stock by the case |
| Structures | Buy and sell furniture |
| Hiring | Hire from a list of candidates |
| Staff | Rosters, shifts and wages |
| Ads | Start and stop ad campaigns |
| Upgrades | See and buy unlocks |

Five of them filter their list through `IsUnlocked`, so a product, a structure, a candidate or a campaign that is locked behind progression is simply not there. The desktop filters its own icons the same way, which is why the Ads app does not exist until day 3.

---

## How an app talks to the game

Two entry points, and no polling.

| Function | When it runs |
|---|---|
| `BindApp` | When the app is opened. It rebuilds the whole list |
| `RefreshBalance` | When your balance changes |

`BindApp` clears the rows and refills them, which is what makes reopening an app show the truth after you bought something. `RefreshBalance` only updates the affordability state of the rows that are already there.

Keeping them separate matters: an early version rebuilt the entire list on every balance change, so buying something destroyed and recreated the row you had just clicked.

The balance itself arrives through the store manager's `OnBalanceChanged` dispatcher. The clock is polled once a second, because the clock is the one thing nothing broadcasts per minute.

---

## Why the screens are sharp

Two numbers have to agree for a world screen to look right, and they are easy to get wrong.

**The render target must be at least as big as the screen's footprint in pixels.** A widget drawn at 1024 across but covering 1411 pixels of your monitor is being magnified, and magnified UI looks soft no matter what anti-aliasing you use.

The rule:

```
DrawSize  ≈  the footprint in pixels when you are leaning in
Scale     =  physical width in cm  /  DrawSize
```

**Lay the widget out at a design size, then scale it.** The shipped screens use a `DesignScale` scale box wrapping a `DesignSize` sized box. Without that, changing `DrawSize` re-flows the whole layout instead of just re-rendering it.

Any new screen should start with the same pair.

---

## Reading pose

`ReadPose` is a scene component on `BP_ScreenBase`, and it is where the camera goes when you lean in. It is a component rather than a computed value so you can nudge the reading distance by dragging it in the viewport.

Its distance is picked so the screen fills a sensible part of the frame. The field of view drops by 30 while reading, which is enough to feel like leaning in without reading as a camera move.

Leaning in also does four things you will want if you build a screen of your own, and all four are in the base class:

- Hides the crosshair and the interaction prompt.
- Releases whatever you are carrying.
- Freezes movement and look through the controller's ignore counters.
- Shows the mouse cursor.

Stepping back undoes all four, and there is exactly one exit path, so a third way out would have to go through it.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
