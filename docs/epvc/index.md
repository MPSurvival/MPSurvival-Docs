# 🎙️ Easy Proximity Voice Chat

Welcome to the official documentation for **Easy Proximity Voice Chat** (EPVC), the drop-in proximity voice chat for Unreal Engine.

No OnlineSubsystem. No EOS/Steam voice setup. It runs over your game's **normal replication**, works on **dedicated servers**, and is fully testable in **PIE**. Voice is treated as a real gameplay signal: channels, occlusion through geometry, runtime-swappable config, server & client muting, and Blueprint events.

> 🎯 Designed to be integrated in minutes, not weeks. Add one component, flip one setting, and talk.

---

## 💥 What is Easy Proximity Voice Chat?

EPVC gives every player a **spatialized voice** that other nearby players hear based on distance, world geometry, and channels, without you touching any online backend. Drop the component on your player, decide *when* it transmits (push-to-talk, open mic, trigger zone), and the plugin handles capture, encoding, replication, jitter buffering, and 3D playback.

- ✅ 100% over normal replication, no OnlineSubsystem / EOS / Steam voice
- ✅ Dedicated-server ready & PIE-testable
- ✅ Blueprint-first API
- ✅ Data-Asset configuration, swappable at runtime

---

## ⚙️ Key Features

### Proximity Voice
- Spatialized 3D playback with attenuation
- Built-in jitter buffer for smooth audio over the network
- Works for listen server **and** dedicated server

### Channels
- Speakers talk and listen on a channel
- Listeners only hear speakers on their **own** channel
- Great for teams, radios, lobbies, spectators

### Occlusion
- Voices get **muffled** by world geometry between speaker and listener
- Per **physical material** presets (glass, concrete, wood…)
- Volume + low-pass stack realistically per surface

### Muting
- **Local muting**: mute one player for yourself only
- **Server muting**: block a player (or everyone) for the whole session
- Authority-safe, replicated

### Runtime Config
- All tunables live in a `ProximityVoiceConfig` Data Asset
- Swap presets live with a single call (e.g. enter a "no-voice" zone)

### Blueprint Events & Debug
- Activity, channel, mute and self-voice events
- In-world **debug overlay** (ranges, occlusion lines, live levels)

---

## 📚 Guides

Use the navigation to set things up, in order:

- [Quick Start](setup/quick_start.md) — enable voice and hear your first words
- [Configuration (Data Asset)](guides/configuration.md) — tune capture, playback, latency
- [Channels](guides/channels.md) — separate who hears who
- [Muting](guides/muting.md) — local & server muting
- [Occlusion](guides/occlusion.md) — muffle voices through walls
- [Blueprint Events](guides/events.md) — react to voice in your UI/gameplay
- [Debug Overlay](guides/debug.md) — see what's happening in-world

---

## 📢 Join the Community

Got ideas? Need help? Found a bug?

Join the **Discord server** to connect with other devs, give feedback, or ask for support. Your suggestions directly shape future updates!

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
