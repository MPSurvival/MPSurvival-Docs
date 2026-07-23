# 🐛 Debug Overlay

EPVC ships with an in-world debug overlay so you can *see* what the voice system is doing: who's transmitting, their ranges, live levels, and occlusion lines. Perfect for tuning attenuation and occlusion, or diagnosing "why can't I hear them".

!!! note
    The overlay is **development-only**. It's stripped from shipping builds, so you can leave the call in without worrying about it in production.

---

### Turn it on

Call the static **Show Voice Debug** node (category **Proximity Voice Chat → Debug**). It affects **every** voice component at once.

```
Show Voice Debug (Enable = true, Show Text = true, Show Ranges = true, Show Occlusion Lines = true)
```

| Param | Description |
|-------|-------------|
| `Enable` | Master toggle for the overlay. |
| `Show Text` | Floating text over each speaker (state, level, channel, occlusion…). |
| `Show Ranges` | Visualizes attenuation ranges around speakers. |
| `Show Occlusion Lines` | Draws one line per emitter (the speaker's body, and every [receiver](receivers.md) rebroadcasting them) toward the listener. |

!!! tip
    Bind it to a debug key (e.g. a dev-only input) so you can toggle the overlay live while walking around your map.

---

### Reading the emitter lines

When a speaker is heard from several places at once (their body **and** a radio), you get **one line per emitter**. Line colour goes from green (clear) to red (occluded), and the **brightness follows the emitter selection**: the brightest line is the one you're actually hearing.

Walk between a speaker and a radio rebroadcasting them and you'll see the handover happen live, which is the fastest way to tune `Emitter Switch Hysteresis` and `Emitter Crossfade Time`. Receiver lines are thicker than body lines and, with `Show Text` on, are labelled with the receiver's actor, its channel and its current weight in percent. See [Radios & Receivers](receivers.md).

---

### What to look for

- **No text over a speaker who should be talking?** They're probably not calling `Set Active Voice(true)`, or they're on a different [channel](channels.md).
- **Text shows transmitting but you hear nothing?** Check [muting](muting.md) and your attenuation range, or you're testing with a single player (you never hear yourself).
- **Voice cuts through walls?** Occlusion lines will show whether the trace hits your geometry, verify the mesh responds to the `Occlusion Trace Channel`.
- **A radio stays silent?** Check the receiver's `Listen Channel` matches the speaker, that `Receiver Enabled` is on, and that its `Max Simultaneous Speakers` isn't already full.

---

## 📢 Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
