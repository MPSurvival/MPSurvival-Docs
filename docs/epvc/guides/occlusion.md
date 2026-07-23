# 🧱 Occlusion

Occlusion makes a voice **muffled** when world geometry sits between the speaker and the local listener. Walk behind a wall and the other players fade and get muddy, exactly what you'd expect.

It's done **per physical material**: glass muffles differently than concrete. Surfaces between speaker and listener **stack**.

---

### How it works

On each listener's machine, EPVC periodically traces from the speaker to the listener and collects the surfaces in between. For every surface hit:

- **Volume multipliers stack** (multiplicatively) — more walls = quieter.
- The **most muffled** low-pass wins — the thickest surface sets the muddiness.

The result is blended in/out smoothly so it never pops.

!!! note
    Occlusion is evaluated **per emitter**, from where each one actually sits. A [radio](receivers.md) behind a wall is muffled even when the speaker is standing in the open, and each emitter uses the config it plays on (the receiver's `Output Configuration` when it has one).

---

### 1. Enable it on the config

In your `ProximityVoiceConfig` asset:

| Property | Description |
|----------|-------------|
| `Enable Occlusion` | Master switch for muffling through geometry. |
| `Occlusion Trace Channel` | Trace channel used to collect surfaces. Set occluding meshes to **Overlap** (or Block) on this channel. Defaults to `Visibility`. |
| `Occluded Volume Multiplier` | Fallback volume kept through a surface with **no matching material** (1 = unchanged, 0 = silent). |
| `Occluded Low Pass Frequency` | Fallback low-pass cutoff (Hz) for an unmatched surface. Lower = more muffled. |
| `Occlusion Interp Speed` | How fast the muffle blends in/out. Higher = snappier. |
| `Occlusion Trace Interval` | Seconds between occlusion traces. Lower = more responsive, more cost. |

---

### 2. Author per-material presets

`Occlusion Materials` is a list of presets, each keyed to a **Physical Material**:

| Property | Description |
|----------|-------------|
| `Physical Material` | The surface this entry matches. Assign the same physical material to your glass/concrete/etc. meshes. |
| `Volume Multiplier` | Volume kept passing through this surface (1 = unchanged, 0 = silent). Stacks multiplicatively. |
| `Low Pass Frequency` | Low-pass cutoff (Hz) through this surface. Lower = more muffled. Most muffled surface wins. |

Any surface **not** in the list uses the `Occluded Volume Multiplier` / `Occluded Low Pass Frequency` fallbacks above.

---

### 3. Set up your world

1. Create/assign **Physical Materials** to the meshes you want to occlude (e.g. `PM_Glass`, `PM_Concrete`).
2. Make sure those meshes respond to the **Occlusion Trace Channel** (Overlap or Block).
3. Add matching entries in the config's `Occlusion Materials`.

!!! tip
    Thin glass → high `Volume Multiplier` (stays loud) but a mid `Low Pass Frequency` (a little muffled). Thick concrete → low `Volume Multiplier` and low `Low Pass Frequency` (quiet and muddy).

!!! note
    You can read how muffled a speaker currently is with **Get Occlusion Amount** on their component (0 = clear, 1 = fully occluded), handy for a UI indicator or the [Debug Overlay](debug.md).

---

## 📢 Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
