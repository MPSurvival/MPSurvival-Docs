# The day/night cycle

The sun follows the store clock. There is no separate time of day, no weather, and nothing to synchronise.

---

## How it works

`BP_DayNightCycle` is an actor you place in the level. Fill in its `Sun` and `Moon` references with the eyedropper in the Details panel and it does the rest.

It reads `GetClockHours()` from the store manager, which is the continuous decimal hour, and turns it into a rotation:

```
Pitch = −180 × ( Hours − SunriseHour ) / ( SunsetHour − SunriseHour )
```

At sunrise the sun is on the horizon, at midday it is overhead, at sunset it is on the opposite horizon.

| Field | What it does | Shipped default |
|---|---|---|
| `SunriseHour` | When the sun comes up | `6` |
| `SunsetHour` | When it goes down | `18` |
| `MoonYaw` | Which way the moon rises from | |
| `MoonPeakElevation` | How high it climbs | |

The actor is the **only** thing that writes the sun's rotation. Everything else about the sky follows: **Atmosphere Sun Light** is ticked on the directional light and the sky light is in **Real Time Capture**, so the horizon colour, the fog and the sky all move on their own.

Because it reads the clock rather than ticking its own timer, it **stops when the clock stops**. Freeze the clock at closing time and the sun stops with it, with nothing written to make that happen.

---

## The sun's intensity is not driven

Worth stating plainly, because it looks like an obvious thing to add: **do not drive `Intensity` on the directional light**.

An intensity curve tied to elevation looks right for one frame and then blackens the sky, because the atmosphere derives the sky colour from the light's intensity as well as its direction. Set the intensity once in the Details panel and only write the rotation.

---

## The moon

The moon is a directional light plus a sprite, locked 180 degrees from the sun. Its own yaw and peak elevation are separate, so it crosses the sky on a believable arc rather than retracing the sun's path with an offset.

Its light is dim, around 0.15 lux at 12000 K. It is there so the store is not pitch black at night, not so you can read by it.

The sprite is a real photograph from NASA's Scientific Visualization Studio, public domain, so it can be redistributed. `M_Moon` draws it unlit, translucent and two sided, with fog and cloud fog turned off.

Two things about it that are deliberately wrong, and one that is deliberately right:

- **It is always full.** A real moon moves about 13 degrees a day relative to the sun, which is what produces phases. Locking it opposite the sun means it is always full. Two played days are fourteen real minutes apart, so nobody compares.
- **It is never visible during the day.** A full moon rises at sunset by definition, so showing one at noon would be more wrong than leaving it out.
- **Its motion across a night is correct.** It travels the same arc at the same rate as the sun does across a day.

If you want real phases, it is a `MoonPhaseOffset` advancing about 12.2 degrees a day, plus a terminator mask in `M_Moon`.

---

## Night

Night is made at the **exposure**, not by turning lights down. With auto exposure, dimming lights just makes the eye adapt and you get a grey night instead of a dark one.

The street lamps come on with the clock. `BP_StreetLamp` reads the same hour as everything else.

Two lighting notes for a level of your own:

- `Cast Shadows` stays on for the moon light. Turning it off would save shadow maps and let moonlight pass through walls and light the shop interior at night, which is far more visible than its cost.
- Forward shading priority is set so the **sun** is the light that affects translucency and volumetric fog. At night the sun is below the horizon so they receive almost nothing, which is the right result. Giving the moon priority would light the fog at midday from a 0.5 lux source.

---

## Timing

At the shipped values a day is 14 real minutes: 08:00 to 22:00 at 60 real seconds per game hour.

The sun sets at 18:00, so the last four hours are played at dusk and then at night. If you want the shop to close in daylight, move `SunsetHour` rather than `ClockStopHour`, since the clock hour is also the pace of the whole day.

---

## What it does not do

No weather. No rain, no overcast, no wind. Adding one is a whole system, and it is not in the template.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
