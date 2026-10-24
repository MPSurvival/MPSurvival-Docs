# Easy Proximity Voice Chat

Welcome to the official documentation for **Easy Proximity Voice Chat** (EPVC), the drop-in proximity voice chat for Unreal Engine.

No OnlineSubsystem. No EOS/Steam voice setup. It runs over your game's **normal replication**, works on **dedicated servers**, and is fully testable in **PIE**. Voice is treated as a real gameplay signal: overlapping channels, radios, voice effects, occlusion through geometry, microphone selection, runtime-swappable config, server & client muting, and Blueprint events.

> 🎯 Designed to be integrated in minutes, not weeks. Add one component, flip one setting, and talk.

---

## What's new in 1.3

- **Overlapping channels**: a player can be on several channels at once, and talk, listen or both on each one. Keep everyone on channel `0` for proximity, give each team a radio channel, and players hear both at the same time. `Set Channel` works as before. See [Channels](guides/channels.md).
- **Microphone selection**: list the microphones and switch between them from your options menu, live, with three static nodes. The choice is saved between sessions. See [Microphone Selection](guides/input_device.md).
- **Lighter voice packets**: each packet is 7 bytes smaller, and a player who talks on no channel sends nothing.

Previously, **1.2** added the voice [Sound Class](guides/sound_class.md) and the [Voice Pitch Shift](guides/effects.md) effect, and **1.1** added [Radios & Receivers](guides/receivers.md), [Voice Effects](guides/effects.md), emitter selection, per-emitter occlusion and the [debug overlay](guides/debug.md).

---

## What is Easy Proximity Voice Chat?

EPVC gives every player a **spatialized voice** that other nearby players hear based on distance, world geometry, and channels, without you touching any online backend. Drop the component on your player, decide *when* it transmits (push-to-talk, open mic, trigger zone), and the plugin handles capture, encoding, replication, jitter buffering, and 3D playback.

- ✅ 100% over normal replication, no OnlineSubsystem / EOS / Steam voice
- ✅ Works **with** Steam and EOS out of the box, zero extra setup
- ✅ Dedicated-server ready & PIE-testable
- ✅ Blueprint-first API
- ✅ Data-Asset configuration, swappable at runtime

---

## Steam & EOS Compatibility

**Yes, EPVC works with Steam and EOS.** To be clear about what "no OnlineSubsystem" means: EPVC replaces their *voice* stack, not their *networking*. You keep using Steam or EOS for sessions, matchmaking and transport exactly as you do today.

This works because EPVC only ever speaks the language of standard Unreal replication — a `Server` RPC to send captured audio, a `NetMulticast` RPC to distribute it. Steam and EOS plug in one layer below that, at the **NetDriver / SocketSubsystem** level (`SteamSockets`, `EOSNetDriver`). Your voice packets ride over whatever transport is active without ever knowing about it.

The result is that the exact same code path runs everywhere:

| Setup | Works | Notes |
|---|---|---|
| PIE / standalone | ✅ | No online backend needed |
| `IpNetDriver` (LAN, direct IP) | ✅ | |
| Steam listen server (P2P) | ✅ | No `bHasVoiceEnabled`, no Steam voice config |
| Steam dedicated server | ✅ | |
| EOS listen server (P2P / relay) | ✅ | No EOS RTC / Voice product required |
| EOS dedicated server | ✅ | |

You do **not** need to enable Steam voice, provision an EOS RTC (Voice) product, or add `OnlineSubsystemSteam` / `OnlineSubsystemEOS` to your dependencies for EPVC to function.

---

## Key Features

### Proximity Voice
- Spatialized 3D playback with attenuation
- Built-in jitter buffer for smooth audio over the network
- Works for listen server **and** dedicated server

### Channels
- A player can be on **several channels at once**, talking, listening or both on each
- A listener hears a speaker who talks on a channel they listen to
- Proximity and a team radio at the same time, with nothing to switch
- Teams, radio push-to-talk, lobbies, spectators, dead players

### Radios & Receivers
- One component turns any actor into a **radio, intercom or PA speaker**
- Rebroadcasts a whole channel with its **own** attenuation, volume and effects
- Hear the channel without being on it, no replication needed on the actor
- Smart emitter selection when a voice reaches you from body **and** radio

### Voice Effects
- Per-speaker **Source Effect Chain** (bitcrusher, filter, EQ, ring mod…)
- Included **Voice Pitch Shift** effect, with per speaker random detune
- Applied per emitter: clean on the pawn, crunchy on the radio
- Swappable at runtime like any other config setting

### Options Menu
- Route the voice through your own **Sound Class**
- Sound Mixes, class volume and ducking apply live, even mid sentence
- Receivers can carry a different class, one slider for voice, one for radio
- **Microphone selection**: list the microphones, switch live, saved between sessions

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

## Guides

Use the navigation to set things up, in order:

- [Quick Start](setup/quick_start.md), enable voice and hear your first words
- [Configuration (Data Asset)](guides/configuration.md), tune capture, playback, latency
- [Channels](guides/channels.md), decide who hears who, on one channel or several
- [Radios & Receivers](guides/receivers.md), rebroadcast a channel out of a radio or intercom
- [Voice Effects](guides/effects.md), bitcrusher, filters, pitch shift and the radio crunch
- [Sound Class & Volume](guides/sound_class.md), wire the voice to your options menu
- [Microphone Selection](guides/input_device.md), let players pick their microphone
- [Muting](guides/muting.md), local & server muting
- [Occlusion](guides/occlusion.md), muffle voices through walls
- [Blueprint Events](guides/events.md), react to voice in your UI/gameplay
- [Debug Overlay](guides/debug.md), see what's happening in-world

---

## Join the Community

Got ideas? Need help? Found a bug?

Join the **Discord server** to connect with other devs, give feedback, or ask for support. Your suggestions directly shape future updates!

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
