# 🔊 Radios & Receivers

A **receiver** rebroadcasts a whole channel out of the actor it sits on: a radio, an intercom, a PA speaker, a walkie-talkie in someone's hand.

Drop the **Proximity Voice Receiver** component on any actor, set its **Listen Channel**, and every speaker talking on that channel is also heard **from that actor**, spatialized from there with its own attenuation, volume and effects. Nothing on other channels comes out of it.

Speakers keep emitting from their own body too, so someone standing next to the speaker hears them directly, and someone standing next to the radio hears them through the radio.

---

### 1. Add a receiver

1. Open the actor Blueprint you want to turn into a radio (a prop, a wall intercom, a handheld device).
2. Add the **Proximity Voice Receiver** component.
3. Set **Listen Channel** to the channel it should pick up (see [Channels](channels.md)).

That's it, it works as soon as it's placed.

!!! note
    The actor does **not** need to be replicated. Voice packets already reach every machine, and each machine decides locally what to play. Placing a radio in the level, spawning it client-side, or attaching it to a hand all work the same way.

---

### 2. Properties

| Property | Description |
|----------|-------------|
| `Listen Channel` | Only speakers on this channel come out of this receiver. Everything else is ignored. |
| `Receiver Enabled` | Turn the receiver on or off. Off is silent and releases its audio immediately. |
| `Output Configuration` | A `ProximityVoiceConfig` asset used for voices coming out of **this** receiver: attenuation, volume, sound class, source effects, occlusion. Leave it empty to use each speaker's own config. |
| `Volume Multiplier` | Volume applied on top of the output configuration. Handy to reuse one config asset at several levels. |
| `Attach Socket` | Mesh socket the voice plays from (e.g. a speaker grille). Falls back to the actor root. |
| `Emitter Priority` | Used when the speaker's `Emitter Selection` is **Priority**. Defaults to `1`, a speaker's own body defaults to `0`. |
| `Max Simultaneous Speakers` | Hard cap on how many speakers can come out of this receiver at once (1 to 16, default `4`). |

!!! note
    Capture and jitter buffer settings are **ignored** in the `Output Configuration`, those belong to the speaker. Only playback-side settings apply here.

---

### 3. Make it sound like a radio

This is the whole point of `Output Configuration`: the same voice can be **clean on the pawn** and **crunchy on the radio**.

1. Create a second `ProximityVoiceConfig` asset, e.g. `DA_RadioReceiver`.
2. Give it a **short** `Proximity Attenuation` (a radio should only be heard in the room it's in).
3. Assign a **Source Effect Chain** with your radio crunch, see [Voice Effects](effects.md).
4. Optionally give it its own `Voice Sound Class`, e.g. `SC_Radio`, so radios get their own volume slider, see [Sound Class & Volume](sound_class.md).
5. Assign that asset to the receiver's `Output Configuration`.

!!! tip
    Occlusion is evaluated **per emitter**, from where each one actually sits. A radio behind a wall is muffled even when the speaker is standing in the open. See [Occlusion](occlusion.md).

---

### 4. When you can hear both at once

If you're close to the speaker **and** close to a radio rebroadcasting them, the same voice reaches you from two places. Playing both is not "twice as loud", it's one signal arriving at two distances, which comb-filters and sounds hollow.

That's what **Emitter Selection** on the speaker's `ProximityVoiceConfig` decides:

| Mode | What you hear |
|------|---------------|
| `Loudest Wins` (default) | The emitter that reaches you loudest, the others fade out. |
| `Priority` | The audible emitter with the highest priority, whatever the distance. Loudness breaks ties. |
| `Mix All` | Every emitter plays. Expect comb filtering when two are audible at once. |

Two more dials go with it, both on the speaker's config:

| Property | Description |
|----------|-------------|
| `Emitter Crossfade Time` | Seconds to fade between emitters when the winner changes. `0` = hard switch (audible click). |
| `Emitter Switch Hysteresis` | How much louder a challenger must be to take over (`0.1` = 10%). Stops the two swapping every frame at the crossover point. |
| `Body Emitter Priority` | Priority of the speaker's own body, used in `Priority` mode. |

!!! tip
    Want the radio to always win, even when you're standing next to the speaker? Set the speaker config's `Emitter Selection` to **Priority** and give the receiver a higher `Emitter Priority` than `Body Emitter Priority`.

---

### 5. Events & getters

On the **Proximity Voice Receiver** component:

| Node | Fires / returns |
|------|-----------------|
| `On Speaker Started` | A speaker started coming out of this receiver. Gives you the `Speaker` actor. |
| `On Speaker Stopped` | A speaker stopped coming out of this receiver. |
| `On Receiver Activity` | Loudest voice on this receiver, every time it moves. `0` when silent. |
| `Is Receiving Voice()` | True while at least one speaker is transmitting on this receiver's channel. |
| `Get Received Voice Level()` | Loudest voice currently on this receiver's channel, `0` when silent. |
| `Get Speakers On Air()` | The actors currently transmitting on this receiver's channel. |
| `Set Listen Channel(int)` | Retune the receiver. Takes effect immediately, including mid-sentence. |
| `Set Receiver Enabled(bool)` | Turn it on or off at runtime. |

From the speaker side, on the **Proximity Voice Chat** component:

| Node | Returns |
|------|---------|
| `Get Active Receivers()` | Receivers currently rebroadcasting this speaker on the local machine. |
| `Get Receiver Audio Component(Receiver)` | The Audio Component this speaker plays through on that receiver. Null while it isn't rebroadcasting them. |

And on the **Proximity Voice Chat Subsystem**:

| Node | Returns |
|------|---------|
| `Get Receivers On Channel(int)` | Every receiver in the level currently rebroadcasting that channel. |

---

### Example: a walkie-talkie with a blinking light

1. Put a **Proximity Voice Receiver** on your walkie-talkie actor, `Listen Channel = 1`.
2. Assign `DA_RadioReceiver` as its `Output Configuration` (short attenuation + crunch chain).
3. Bind **On Receiver Activity** and drive a light / material parameter with `Volume`.
4. Bind **On Speaker Started** / **On Speaker Stopped** to play a *click* or a burst of static.
5. Use **Set Receiver Enabled** for an on/off switch, and **Set Listen Channel** for a frequency knob.

---

### Good to know

- A speaker **never** hears their own voice come back out of a receiver, no feedback loop to worry about.
- Receivers only hold a speaker while that speaker is **actually transmitting**. Slots are freed at the end of each talk burst.
- Slots are first come, first served. A speaker who finds the receiver full is simply not rebroadcast, and still emits from their own body.
- A speaker is rebroadcast even when the local listener is on **another channel**, that's the point of a radio.
- [Muting](muting.md) still applies. A locally muted or server-muted player doesn't come out of a receiver either.

---

## 📢 Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
