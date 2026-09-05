# The office computer

Management happens on a computer on the desk in the back office, and you walk over to it.

Press `E` and the camera leans in: the crosshair and prompt go away, whatever you were carrying is put down, and the mouse cursor appears. From there it is a desktop — icons, a window with a title bar, a taskbar with the time on it. `Tab` or `E` leans you back out.

---

## The desktop

`BP_Computer` holds one thing you edit: `Apps`, an array of `BP_AppDataAsset` in its Details panel. It draws an icon for each entry, in the order of the array, and opens the widget class the asset names.

**To add or remove an app on a computer**, edit that array. See [Write your own app](add_an_app.md).

The chrome is three bands:

| Band | Height | Contents |
|---|---|---|
| Title bar | 64 | App icon, app name, your balance, a close button |
| App area | 960 | Whatever the app draws |
| Taskbar | 56 | Time and day |

The title bar takes the app's own colour, and that is the only accent in the interface. The close button closes the **app**, not the computer — you leave the computer by stepping back.

---

## The mouse

It is the real cursor, so hover, click, scroll and drag all behave as you expect, with nothing wired.

Two things to copy if you build a screen of your own, both checkboxes, both already set on the shipped screens:

- **Buttons on a world screen must be non-focusable**, or `Tab` walks a focus rectangle around them instead of leaving the screen.
- **The widget component's window must not be focusable either**, or it swallows keyboard input.

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

Five of them hide what is still locked behind progression, and the desktop hides its own icons the same way, which is why the Ads app does not appear until day 3. See [Unlocks](../progress/unlocks.md).

---

## The reading pose

`ReadPose` is a scene component on `BP_ScreenBase`: it is where the camera goes when you lean in, so you can nudge the reading distance by dragging it in the viewport. The field of view drops by 30 while reading.

Leaning in also hides the crosshair and the prompt, releases whatever you are carrying, freezes movement and look, and shows the cursor. Stepping back undoes all four. Any screen you build on `BP_ScreenBase` gets that for free — see [Screens in the world](../ui/diegetic_screens.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
