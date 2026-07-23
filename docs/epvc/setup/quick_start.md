# 🚀 Quick Start

Get from zero to "I can hear my friend" in a few minutes.

---

### 1. Enable voice in your project

The engine's voice capture is **off by default**. Open `Config/DefaultEngine.ini` and add:

```ini
[Voice]
bEnabled=true
```

!!! warning
    This value is read at **engine startup**. After adding it, **fully restart the editor** — it won't take effect on a hot reload.

Your microphone is normally **already allowed** by the OS, so you shouldn't have to touch anything. But if you run into trouble and no capture starts, come back and double-check it here (Windows: **Settings → Privacy → Microphone**).

---

### 2. Add the component to your player

Open your player **Character/Pawn** Blueprint and add the **Proximity Voice Chat** component.

!!! tip
    The owning actor should be **replicated** and **player-controlled** (a normal networked Character/Pawn). The component is the *speaker*: whatever actor it lives on is what other players hear.

Optionally assign a **Default Configuration** on the component (a `ProximityVoiceConfig` Data Asset). If you leave it empty, sensible built-in defaults are used. See [Configuration](../guides/configuration.md).

---

### 3. Decide when to transmit

EPVC never guesses *when* you want to talk, your game decides. Call **Set Active Voice** on the component:

- **Push-to-talk**: `Set Active Voice(true)` on key pressed, `Set Active Voice(false)` on released.
- **Open mic**: `Set Active Voice(true)` once at BeginPlay.
- **Trigger / radio**: toggle it from your own gameplay logic.

!!! note
    Call `Set Active Voice` on the **owning client** (the player who is talking). Transmission is replicated for you.

---

### 4. Test it

Play in the editor with **at least 2 players**:

- **Number of Players**: `2`
- **Net Mode**: `Play As Listen Server` (or `Play As Client` against a dedicated server)

Walk the two players close together and talk.

!!! warning
    Proximity voice does **not** play back on your *own* machine, you can't hear yourself. You need a second player (or a second PIE window) to hear anything.

---

### ✅ Checklist if you hear nothing

| Check | Why |
|-------|-----|
| `[Voice] bEnabled=true` added **and editor restarted** | Capture won't init otherwise |
| Microphone allowed by the OS | No mic = no capture |
| `Set Active Voice(true)` is actually being called | Nothing is sent until you transmit |
| Testing with **2+ players** | You never hear yourself |
| Both players on the **same channel** | Listeners only hear their own channel — see [Channels](../guides/channels.md) |
| Speaker not **server-muted** | See [Muting](../guides/muting.md) |

!!! tip
    Turn on the [Debug Overlay](../guides/debug.md) to *see* who's transmitting, their ranges, and occlusion in real time.

---

### 5. Going further

Once you can hear each other, the rest is opt-in:

- [Channels](../guides/channels.md), so teams don't hear each other
- [Radios & Receivers](../guides/receivers.md), one component to turn a prop into a radio or an intercom
- [Voice Effects](../guides/effects.md), the radio crunch and other source effects
- [Occlusion](../guides/occlusion.md), muffle voices through walls

---

## 📢 Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
