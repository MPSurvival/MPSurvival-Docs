# Microphone Selection

Let players pick their microphone from your options menu. The new microphone is used at once, mid-sentence included, and EPVC saves the choice, so it is back on the next launch.

---

### The nodes

The three nodes are static. Call them from any Blueprint, a Widget Blueprint included: they need no component.

| Node | What it does |
|------|--------------|
| `Get Voice Input Devices()` | The microphones the voice can capture from, system default first. Each entry has a `Device Name` and `Is System Default`. |
| `Get Voice Input Device()` | The microphone picked with `Set Voice Input Device`. Empty while the voice follows the system default. |
| `Set Voice Input Device(Device Name)` | Capture from this microphone. An empty name follows the system default again. Returns `false` when the microphone cannot be opened, and the current one stays. |

They are under **Proximity Voice Chat > Input Device** in the node menu.

---

### Build the selector

This uses a **Combo Box (String)** in your settings widget, named `MicCombo` here.

1. On **Event Construct**, call **Clear Options** on `MicCombo`.
2. Call `Get Voice Input Devices`, and **Add Option** on `MicCombo` for each entry's `Device Name`.
3. Call `Get Voice Input Device`. If it is empty, select the entry whose `Is System Default` is true. Otherwise, select that name.
4. Bind **On Selection Changed** on `MicCombo`. Ignore the event when `Selection Type` is `Direct`, that is your own step 3.
5. Otherwise, call `Set Voice Input Device` with `Selected Item`.
6. If it returns `false`, select `Get Voice Input Device` again and tell the player the microphone is not available.

!!! tip
    Want a "System default" line in the list? Add it as the first option yourself, and call `Set Voice Input Device` with an empty name when it is picked.

---

### How it behaves

- The choice is for the whole game: every local player captures from the same microphone. In PIE, every window uses it.
- EPVC saves it in `GameUserSettings.ini`, section `[EasyProximityVoiceChat]`, key `InputDevice`. Remove the key to go back to the system default.
- If the saved microphone is unplugged when the game starts, EPVC captures from the system default and writes a warning to the log. The saved choice stays, so the next launch uses that microphone again if it is plugged in.
- Unreal finds the microphones when the game starts. A microphone plugged in later shows in the list, but `Set Voice Input Device` returns `false` for it until the game restarts.
- On **Mac**, Unreal's voice capture always records from the input picked in the macOS **Sound** settings. `Get Voice Input Devices` returns an empty list there: hide the selector when the list is empty.

---

## Need help?

[👉 Join the Discord](https://discord.gg/EqHCtq38jy)

---
