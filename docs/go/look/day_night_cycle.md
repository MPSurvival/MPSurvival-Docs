# The day/night cycle

The sun follows the store clock. There is no separate time of day to set and nothing to synchronise: freeze the clock at closing time and the sun stops with it.

---

## Setting it up

`BP_DayNightCycle` is an actor you place in the level. Fill in its `Sun` and `Moon` references with the eyedropper in the Details panel and it does the rest.

| Field | What it does | Shipped |
|---|---|---|
| `SunriseHour` | When the sun comes up | `6` |
| `SunsetHour` | When it goes down | `18` |
| `MoonYaw` | Which way the moon rises from | |
| `MoonPeakElevation` | How high it climbs | |

At sunrise the sun is on the horizon, at midday overhead, at sunset on the opposite horizon. **Atmosphere Sun Light** is ticked on the directional light and the sky light is in **Real Time Capture**, so the horizon colour, the fog and the sky follow on their own.

!!! warning
    **Do not drive `Intensity` on the directional light.** The atmosphere derives the sky colour from the intensity as well as the direction, so an intensity curve blackens the sky. Set the intensity once in the Details panel and write only the rotation.

---

## The moon

A directional light plus a sprite, locked opposite the sun, with its own yaw and peak elevation so it crosses the sky on its own arc. Its light is dim, around 0.15 lux at 12000 K: enough that the store is not pitch black at night, not enough to read by.

Because it is locked opposite the sun it is always full, and it never appears during the day. If you want phases, it is a `MoonPhaseOffset` advancing about 12.2 degrees a day plus a terminator mask in `M_Moon`.

The sprite is a public domain NASA photograph, so it redistributes with the template.

---

## Night

Night is made at the **exposure**, not by turning lights down: with auto exposure, dimming lights just makes the eye adapt and you get a grey night instead of a dark one.

The street lamps come on with the clock, from the same hour as everything else.

Two notes for a level of your own:

- Leave `Cast Shadows` on for the moon light, or moonlight passes through walls and lights the shop interior at night.
- Forward shading priority is set so the **sun** is the light that affects translucency and volumetric fog. Giving it to the moon would light the fog at midday from a 0.5 lux source.

---

## Timing

At the shipped values a day is 14 real minutes: 08:00 to 22:00 at 60 real seconds per game hour.

The sun sets at 18:00, so the last four hours are played at dusk and then at night. To close the shop in daylight, move `SunsetHour` rather than `ClockStopHour`, since the clock hour is also the pace of the whole day.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
