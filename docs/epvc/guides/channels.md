# Channels

Channels decide **who hears who**. A player can be on **several channels at once**, and on each one they can **talk**, **listen**, or both.

A listener hears a speaker when the speaker **talks** on a channel the listener **listens** to. One shared channel is enough, and a voice never plays twice because two channels match.

Everyone starts on channel `0`, talking and listening. With nothing else set up, that is plain proximity chat.

---

### One channel

**Set Channel** puts the player on one channel only, talking and listening. They leave every other channel.

```
Set Channel (New Channel = 1)
```

This is how channels worked before 1.3, and it still works the same way.

---

### Several channels at once

**Join Channel** adds a channel and keeps the others. A player on `0` and `1` talks on both and hears both.

```
Join Channel (Channel = 1, Talk = true, Listen = true)
```

All the channel nodes are on the **Proximity Voice Chat** component:

| Node | What it does |
|------|--------------|
| `Set Channel(Channel)` | Leave every channel, then talk and listen on this one. |
| `Join Channel(Channel, Talk, Listen)` | Join a channel, or change how you are on it. Your other channels stay. With `Talk` and `Listen` both off, you leave it. |
| `Leave Channel(Channel)` | Leave this channel only. |
| `Set Talking On Channel(Channel, Talk)` | Turn your voice on or off on one channel. What you hear does not change. If you were not on the channel, you join it with `Listen` off. |
| `Get Channels()` | Every channel the player is on, each with its `Talk` and `Listen` flags. |
| `Is Talking On Channel(Channel)` | True if the player's voice goes out on this channel while they transmit. |
| `Is Listening On Channel(Channel)` | True if the player hears the voices sent on this channel. |
| `Get Channel()` | The first channel the player talks on. |

To start a pawn on other channels, set `Channels` on the component in the **Details** panel, under **Proximity Voice Chat > Channels**.

!!! note
    Call these nodes on the **owning client** or on the **server**. Changes replicate, and `On Channel Changed` fires on every machine. A call made on another player's machine does not reach the server.

---

### A channel is not a "hear from anywhere" pass

A voice heard through a channel still comes out of the speaker's **body**, with its attenuation and its occlusion. A teammate on your team channel but 200 meters away is out of earshot.

To hear a channel from anywhere, give the listener a **receiver** tuned to it: a walkie-talkie, a radio, an intercom. A receiver plays everyone who talks on its `Listen Channel`, from where the receiver is, with its own attenuation and effects. See [Radios & Receivers](receivers.md).

---

### Example: proximity and a team radio at the same time

Everyone hears everyone nearby on channel `0`. Team 1 has a walkie-talkie on channel `1`, team 2 on channel `2`.

1. Leave every player on channel `0`.
2. When a player joins a team, call `Join Channel` with the team's radio channel (`1` or `2`) on their voice component.
3. Add a **Proximity Voice Receiver** to the player's character: the walkie-talkie.
4. Set its `Listen Channel` to the same team channel, and give it a radio `Output Configuration` (short attenuation, radio crunch).

When a team 1 player talks, the players close to them hear their voice on channel `0`, enemies included. Every team 1 player also hears it through their walkie-talkie, wherever they are. Nobody has to switch channels: each player hears both at once.

The receiver plays on every machine, so a player standing next to someone else's walkie-talkie hears it too. If only its owner should hear it, turn `Receiver Enabled` off on the characters that are not locally controlled.

!!! tip
    A teammate standing next to you reaches you twice: from their body and from your walkie-talkie. **Emitter Selection** decides which one you hear. With `Loudest Wins`, lower the receiver's `Volume Multiplier` if you want a close teammate heard from their body. See [When you can hear both at once](receivers.md#4-when-you-can-hear-both-at-once).

---

### Example: radio push-to-talk

Same setup, but the voice goes on the team radio only while the radio key is held.

1. When the player joins a team, call `Join Channel(1, Talk = false, Listen = true)`. They hear the radio but do not talk on it yet.
2. On the radio key **pressed**: `Set Talking On Channel(1, true)`, then `Set Active Voice(true)`.
3. On the radio key **released**: `Set Active Voice(false)`, then `Set Talking On Channel(1, false)`.
4. Keep your normal push-to-talk key as it is. It calls `Set Active Voice` only, so that voice stays on channel `0`.

Players near the speaker hear them in both cases, because the speaker still talks on channel `0`.

---

### Example: dead players

Dead players hear the living, but the living do not hear the dead. The dead hear each other.

1. On death, call `Set Channel(5)`. The player now talks and listens on channel `5` only.
2. Then call `Join Channel(0, Talk = false, Listen = true)`. They hear channel `0` again, without talking on it.
3. On respawn, call `Set Channel(0)`.

This needs the voice component to stay alive with the player: keep it on the pawn they spectate with.

---

### Know the local player's channels

From the **Proximity Voice Chat Subsystem** (a World Subsystem, get it with **Get World Subsystem**):

| Node | Returns |
|------|---------|
| `Get Local Listener Channel()` | The first channel the local player listens on. |
| `Is Local Listener On Channel(Channel)` | True if the local player hears the voices sent on this channel. |

Use them for UI ("Team Radio 1"), or to gate other systems.

---

### Good to know

- Channels are plain integers. Pick any numbering that suits your game.
- A player can be on up to 16 channels.
- A player who talks on no channel sends no voice packets at all, so a listen-only spectator uses no voice bandwidth.
- `On Voice Activity` and `On Self Voice Activity` give `Get Channel` as their `Channel`. When a player is on several channels, ask their component with `Is Talking On Channel`.

!!! tip
    Channels and [Occlusion](occlusion.md) work together: teammates on the same channel are still muffled by walls, unless the config they play through has occlusion turned off.

---

## Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
