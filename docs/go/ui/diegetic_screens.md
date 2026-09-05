# Screens in the world

Three screens ship with the template: the checkout display, the card terminal and the office computer. All three are children of `BP_ScreenBase`, and two of them have no graph at all.

---

## Building one

1. Create a child of `BP_ScreenBase`.
2. Set `ScreenMesh` to your monitor mesh.
3. Set `Display.WidgetClass` to your widget, a child of `BP_ScreenBaseWidget`.
4. Position and scale `Display` so it sits on the glass.
5. Move `ReadPose` until leaning in frames the screen the way you want.

`BP_CheckoutScreen` is exactly that and nothing more: two values in the Details panel and an empty graph. It is the one to copy.

---

## What you get from the base

| Component | What it is |
|---|---|
| `ScreenMesh` | The physical screen, the root |
| `Display` | The `WidgetComponent` for the panel |
| `ReadPose` | Where the camera goes when you lean in |

And the behaviour, with nothing to wire: an outline and a prompt, `E` to lean in with the field of view blending, the crosshair and prompt hidden, whatever you were carrying released, movement and look frozen, the mouse cursor shown, `Tab` or `E` to leave, and `SetPowered(false)` to switch the whole thing off.

---

## The widget side

| Function | What to do with it |
|---|---|
| `SetHost(NewHost)` | Called by the screen. Leave it alone |
| `BindHost` | Empty in the base. Override it and subscribe to your host |

There is a matching `BeginRead(Reader)` on the actor, called when someone leans in. Override it if your screen has to wake up rather than simply be looked at.

---

## Sizing the display

Do this with numbers, not by eye — it is what decides whether the screen looks sharp or soft.

```
DrawSize  ≈  how many screen pixels the panel covers when you lean in
Scale     =  physical width in cm  /  DrawSize
```

A widget rendered at 1024 across but covering 1411 pixels on the monitor is being magnified, and no anti-aliasing hides that.

Then lay the widget out at a fixed design size inside a scale box, so changing `DrawSize` re-renders instead of re-flowing your layout.

Two settings on the widget component, and one on every button inside it:

| Setting | Value | Why |
|---|---|---|
| Blend Mode | `Transparent` | `Masked` ghosts under temporal anti-aliasing |
| Window Focusable | off | A focusable window steals keyboard input |
| Is Focusable, on each button | off | Otherwise `Tab` walks a focus rectangle around the screen |

---

## Positioning the panel on the mesh

**Measure the glass, not the bounds.** A monitor's bounding box includes its stand, so the front face of the box is not the front of the screen.

**A real bezel is not symmetrical.** The shipped POS display has 1.6 cm at the top and 3.3 at the bottom; a panel centred in the mesh looks like a laptop, not a till.

Position and scale are Details panel values, so re-exporting the mesh means nudging two fields.

---

## Screen space instead

`BP_DrawerWidget`, the change readout on the till drawer, is in **screen space** rather than world space so it stays readable while you move your head over the drawer. It still belongs to the drawer, and it still lives in the `World/` widget folder.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
