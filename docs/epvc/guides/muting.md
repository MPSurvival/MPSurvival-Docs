# 🔇 Muting

EPVC has two independent muting layers:

- **Local mute** — for *your* machine only. Other players still hear the muted speaker.
- **Server mute** — authoritative. Blocks the speaker's transmission for **everyone**.

---

## 🙉 Local muting (client-side)

Mute a specific player just for yourself. Great for a "mute this player" button in your scoreboard.

Call these on your **own** voice component, passing the **Pawn/Actor** that owns the voice you want to mute:

```
Mute Player Locally (Player To Mute = ThatPawn, Mute = true)
```

| Node | Description |
|------|-------------|
| `Mute Player Locally(PlayerToMute, bMute)` | Mute/unmute one player, locally only. |
| `Is Player Muted Locally(PlayerToMute)` | Query the local mute state. |

!!! note
    Local muting never leaves your machine, no replication, no server involvement. It's purely "I don't want to hear this person".

---

## 🛑 Server muting (authoritative)

Server muting is enforced for the whole session. There are two ways in.

### From a component

| Node | Description |
|------|-------------|
| `Set Server Muted(bMuted)` | **Authority only.** Block/allow this player's transmission for everyone. |
| `Is Server Muted()` | Query whether this player is currently server-muted. |
| `On Server Mute Changed(bMuted)` | Event fired on every machine when the state changes (great for a UI icon). |

`Set Server Muted` is **BlueprintAuthorityOnly**, so run it on the server.

### From the subsystem (moderation hub)

Grab the **Proximity Voice Chat Subsystem** (World Subsystem) for session-wide moderation:

| Node | Description |
|------|-------------|
| `Server Mute Player(PlayerToMute, bMuted)` | Block/allow one player for everyone. Pass their Pawn/Actor. |
| `Server Mute All(bMuted)` | Block/allow **every** player for everyone. |
| `Is All Muted()` | True while `Server Mute All(true)` is in effect. |

```
Get World Subsystem (Proximity Voice Chat Subsystem) → Server Mute Player (PlayerToMute = Cheater, Muted = true)
```

!!! warning
    Anyone can *call* these from Blueprint, so **gate them behind your own admin/permission logic**. The plugin enforces authority, but it doesn't decide *who* is allowed to be a moderator, that's your game's call.

!!! tip
    `Server Mute All(true)` is perfect for cutscenes, round transitions, or a "silence everyone" admin action. Un-mute with `Server Mute All(false)`.

---

## 📢 Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
