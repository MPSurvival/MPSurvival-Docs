# 📡 Blueprint Events

EPVC exposes voice as gameplay signals so you can drive UI and gameplay: speaking indicators, HUD icons, lip-sync triggers, "someone is talking" prompts, and more.

All events live on the **Proximity Voice Chat** component.

---

### Events

| Event | Fires on | Params | Use it for |
|-------|----------|--------|------------|
| `On Voice Activity` | Every machine, for any speaker | `Speaker`, `Volume`, `Channel` | Speaking indicators above other players. Non-zero only while transmitting. |
| `On Channel Changed` | Every machine | `New Channel` | Update team/radio UI. |
| `On Voice Activated` | The speaker's own machine | — | Local "mic live" indicator, start push-to-talk FX. |
| `On Voice Deactivated` | The speaker's own machine | — | Stop the above. |
| `On Self Voice Activity` | Owning client + server only | `Volume`, `Channel` | Your **own** mic level meter. |
| `On Server Mute Changed` | Every machine | `bMuted` | Show a muted icon over a server-muted player. |

!!! tip
    For a **"who is talking" list**, bind `On Voice Activity`: `Speaker` is the owning actor and `Volume` is non-zero only while they transmit. Filter by `Channel` if you only care about your own.

---

### Handy getters

You don't always need events, these are pure reads on the component:

| Node | Returns |
|------|---------|
| `Get Is Active Voice()` | True while this player is transmitting (valid on server and all clients). |
| `Is Self Speaking()` | True while transmitting, as seen on the player's own machine. |
| `Get Voice Level()` | On the owner: live mic level. Elsewhere: last packet level, decaying to 0. |
| `Get Occlusion Amount()` | 0..1 how muffled this speaker is on the local machine. |
| `Is Capture Ready()` | True on the owning client once the mic is initialized. |
| `Get Playback Audio Component()` | The Audio Component the remote voice plays through (null until the first packet). |

---

### Example: floating "speaking" icon

1. On each player widget, bind to `On Voice Activity`.
2. When `Volume > 0`, show the icon; scale/opacity by `Volume` for a nice VU effect.
3. Optionally hide it when `Get Occlusion Amount()` is high (they're behind a wall anyway).

---

## 📢 Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
