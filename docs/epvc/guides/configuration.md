# 🎛️ Configuration (Data Asset)

Every tunable lives in a **`ProximityVoiceConfig`** Data Asset. Author presets, assign one as the component's **Default Configuration**, and swap them at runtime with **Set Voice Configuration**.

---

### 1. Create a config asset

In the **Content Browser**, right-click → **Miscellaneous → Data Asset**, pick **`ProximityVoiceConfig`**, and name it something like `DA_NormalVoice`.

---

### 2. Capture

Controls how the microphone is read and gated.

| Property | Description |
|----------|-------------|
| `Min Decibels` | Mic loudness (dBFS) mapped to Volume `0.0`. Raise toward 0 if background noise keeps the level high. |
| `Max Decibels` | Mic loudness (dBFS) mapped to Volume `1.0`. Lower toward 0 if normal talking reads too quiet. |
| `Mic Silence Detection Threshold` | Envelope (0..1) below which nothing is captured. Lower it for soft speech. |
| `Mic Noise Gate Threshold` | Envelope below which the mic is gated toward silence. Lower for soft speakers. |
| `Mic Input Gain` | Linear gain applied before gating. Raise to boost a quiet mic. |

---

### 3. Playback

How the voice sounds on receiving machines.

| Property | Description |
|----------|-------------|
| `Output Volume` | Playback volume on receiving machines. |
| `Proximity Attenuation` | Sound Attenuation used for spatialized playback. If empty, engine defaults are used. |
| `Playback Attach Socket` | Mesh socket the voice plays from (e.g. `head`). Falls back to the actor root. |
| `Base Low Pass Frequency` | Always-on low-pass cutoff (Hz) for this speaker, independent of occlusion. `0` = off. |

!!! tip
    Assign a **Sound Attenuation** asset to `Proximity Attenuation` to control how far a voice carries and how it falls off with distance. This is your main "how loud, how far" dial.

---

### 4. Activity

| Property | Description |
|----------|-------------|
| `Voice Activity Change Threshold` | Minimum change in Volume (0..1) before `OnVoiceActivity` fires again. |

See [Blueprint Events](events.md) for what to do with activity.

---

### 5. Occlusion

Muffling through world geometry has its own page: see [Occlusion](occlusion.md) for `bEnableOcclusion`, `Occlusion Materials`, and the fallback values.

---

### 6. Jitter Buffer

Absorbs network jitter by buffering a little audio before playback.

| Property | Description |
|----------|-------------|
| `Prime Latency Ms` | Audio buffered before playback starts, per talk burst. Lower = snappier, higher = safer. Clamped to ≤ `Target Latency Ms`. |
| `Target Latency Ms` | Target audio kept buffered while playing. Higher = smoother but more latency. |
| `Max Latency Ms` | Hard cap on buffered audio. Anything beyond is dropped so latency can't run away. |

!!! note
    For LAN or a strong connection you can lower these for tighter latency. For a rough connection, raise `Target Latency Ms` to reduce dropouts.

---

### 7. Swap configs at runtime

Call **Set Voice Configuration** on the speaker's component to change presets live, for example a "muffled radio" preset, or a **no-voice** preset inside a trigger zone.

```
Set Voice Configuration (New Configuration = DA_NoVoice)
```

!!! warning
    The active config is **replicated**. Set it **once on the server** (authority) and it propagates to everyone. If you drive it from a Level Blueprint trigger, gate the call behind **Switch Has Authority → Authority**, otherwise every client fires it redundantly and fights the replicated value.

---

## 📢 Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
