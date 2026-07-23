# 🎚️ Voice Effects

Voices can run through a **Source Effect Chain**: bitcrusher, filter, EQ, ring modulation, whatever you assemble. That's how you get a radio crunch, a helmet filter, a possessed-monster voice, or a phone call.

It lives on the `ProximityVoiceConfig` asset, so it follows the same rules as everything else: author presets, assign, swap at runtime.

---

### 1. Build the chain

In the **Content Browser**:

1. Right-click → **Audio → Effects** and create one or more **Source Effect Presets** (Bit Crusher, Filter, EQ, Ring Modulation, Waveshaper...).
2. Tune each preset by double-clicking it.
3. Right-click → **Audio → Effects → Source Effect Preset Chain** and add your presets to its `Chain` array, in the order you want them applied.

!!! warning
    Voice is **mono**. Use effects that support **1 channel**. Stereo-only effects (stereo delay, panner, stereo widening) will not behave as expected on a voice source.

---

### 2. Assign it

In your `ProximityVoiceConfig` asset, under **Playback**:

| Property | Description |
|----------|-------------|
| `Source Effect Chain` | Source effects applied to the voice itself, per speaker and per emitter. |

!!! note
    The chain is picked up on the **next talk burst**. Swapping it while someone is mid-sentence won't change that sentence, the next one will use it.

---

### 3. Clean on the body, crunchy on the radio

The chain is applied **per emitter**, not per speaker. Since a [receiver](receivers.md) can override the config with its own `Output Configuration`, the very same voice can come out clean from the player's mouth and processed from a radio, at the same time, on the same machine.

The usual setup:

| Asset | Used by | Chain |
|-------|---------|-------|
| `DA_NormalVoice` | The speaker's `Default Configuration` | None, or a light one |
| `DA_RadioReceiver` | The receiver's `Output Configuration` | Bitcrusher + band-pass, the radio crunch |

See [Radios & Receivers](receivers.md) for the receiver side.

---

### 4. A radio crunch that works

A convincing radio is mostly **bandwidth**, not distortion:

1. **Filter** (band-pass): cut everything below ~300 Hz and above ~3 kHz. That alone already sounds like a speaker grille.
2. **Bit Crusher**: drop the sample rate / bit depth a little for digital grit. Go easy, too much and speech becomes unintelligible.
3. Optionally a touch of **Waveshaper** for saturation.

!!! tip
    Just need a muffled voice, no chain at all? `Base Low Pass Frequency` on the same config gives you an always-on low-pass for free. See [Configuration](configuration.md).

---

### 5. Swap effects at runtime

The chain is part of the config asset, so **Set Voice Configuration** swaps it like everything else: put on a gas mask, switch to `DA_MaskedVoice`, take it off, switch back.

```
Set Voice Configuration (New Configuration = DA_MaskedVoice)
```

!!! warning
    The active config is **replicated**. Set it once on the server (authority) so everyone hears the same thing. See [Configuration](configuration.md).

---

## 📢 Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
