# 🔊 Sound Class & Menu Volume

Proximity voice is not a normal `Sound Wave`, it's a live stream, so Unreal gives you no obvious place to plug it into your audio settings. EPVC does: pick a **Sound Class** on the config asset and the voice becomes a classed sound like any other, so **Sound Mixes**, class volume and ducking all apply to it.

That's what lets you ship a **Voice** slider next to Master, Music and SFX.

---

### 1. Create the assets

In the **Content Browser**:

1. Right-click → **Audio → Classes → Sound Class**, name it `SC_Voice`.
2. Right-click → **Audio → Classes → Sound Mix**, name it `SMIX_Options`. This is the mix your settings menu will drive.

!!! tip
    Parent `SC_Voice` under your existing `SC_Master` (open `SC_Master` and drag a connection) and the master slider will move the voice too, for free.

---

### 2. Assign it

In your `ProximityVoiceConfig` asset, under **Playback**:

| Property | Description |
|----------|-------------|
| `Voice Sound Class` | Sound Class the voice plays through. Empty = unclassed, the voice ignores your mixes. |

!!! note
    The class is read when a source **starts**, so a config swapped mid-sentence lands on the **next talk burst**. Same rule as `Proximity Attenuation` and `Source Effect Chain`.

---

### 3. Drive it from your options menu

A Sound Class on its own changes nothing, it only labels the sound. What moves the volume is either the class's own `Volume` property, or a **Sound Mix** targeting it. The second one is what a settings slider should use.

Once, at startup:

```
Push Sound Mix Modifier (In Sound Mix Modifier = SMIX_Options)
```

Then on every slider change:

```
Set Sound Mix Class Override
    In Sound Mix Modifier = SMIX_Options
    In Sound Class        = SC_Voice
    Volume                = <your slider value>
    Pitch                 = 1.0
    Fade In Time          = 0.1
    Apply To Children     = false
```

To drop back to the class defaults, **Clear Sound Mix Class Override** with the same mix and class.

!!! warning
    The mix has to be **pushed** for the override to do anything. If your slider seems dead, that missing `Push Sound Mix Modifier` is the usual reason.

!!! note
    This is **local**, it only affects the machine running it, which is exactly what you want for a settings menu. Nothing here is replicated.

Volume changes apply live, even while someone is mid-sentence, so you can drag the slider and hear it move.

---

### 4. Separate voice and radio sliders

The class comes from whichever config an emitter runs on, and a [receiver](receivers.md) can override the config with its own `Output Configuration`. Give that one a different class and the same speaker feeds two different sliders at once.

| Asset | Used by | Sound Class |
|-------|---------|-------------|
| `DA_NormalVoice` | The speaker's `Default Configuration` | `SC_Voice` |
| `DA_RadioReceiver` | The receiver's `Output Configuration` | `SC_Radio` |

The player then hears the same person at full volume from their mouth and at their own radio volume through the intercom, controlled independently.

---

### 5. Quick sanity check

Not sure the class is really applied? Open `SC_Voice`, set its `Volume` to `0.15`, and talk. If the voice drops, the routing is good, put it back to `1.0` afterwards. This costs no Blueprint at all and rules out a wiring mistake in your menu.

!!! note
    A player never hears their own voice, so run the check on the machine that **listens**, not the one that talks.

---

## 📢 Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
