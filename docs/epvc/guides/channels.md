# 📻 Channels

Channels decide **who hears who**. A listener only hears speakers on their **own** channel. Everyone starts on channel `0`.

Use them for teams, radio frequencies, lobby vs in-game, spectators, proximity "rooms", anything where a subset of players should share voice.

---

### Set a player's channel

Call **Set Channel** on the component. The player will now **talk and listen** on that channel.

```
Set Channel (New Channel = 1)
```

- `Get Channel` → the component's current channel.
- Changing channel replicates, and fires `On Channel Changed` on every machine.

!!! note
    A component's channel is used **both** for transmitting and for listening. If you set a player to channel `2`, they broadcast on `2` and only hear others on `2`.

---

### Know the local listener's channel

From the **Proximity Voice Chat Subsystem** (a World Subsystem, grab it with **Get World Subsystem**):

```
Get World Subsystem (Proximity Voice Chat Subsystem) → Get Local Listener Channel
```

Handy for UI ("You are on Team Radio 1") or gating other systems.

---

### Example: team voice

1. When a player joins a team, `Set Channel(TeamIndex)` on their voice component (on the server / owner).
2. All teammates share the same channel → they hear each other.
3. Enemies on a different channel are silent to them.

!!! tip
    Combine channels with [Occlusion](occlusion.md): teammates on the same channel still get muffled by walls, so a channel isn't a magic "hear through everything" pass unless you want it to be (give that config a disabled occlusion preset).

---

## 📢 Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
