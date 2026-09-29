# The controls, and adding an input

Input goes through **Enhanced Input**. The gameplay keys live in one mapping context, `Content/MPFriendslop/Inputs/IMC_Default`, and the actions live next to it in `Inputs/Inputs/`. `BP_FriendslopPlayerController` adds `IMC_Default` for the local player, through its `Default Context` field, at priority `0`.

Two smaller contexts sit next to it: `IMC_EmoteWheel`, added while the emote wheel is open, and `IMC_Spectator`, added while you spectate after dying.

---

## The keys that ship

| Action | Key | `IA_` asset |
|---|---|---|
| Move | `W` `A` `S` `D` | `IA_Move` |
| Look | mouse | `IA_Look` |
| Sprint (hold) | `Left Shift` | `IA_Sprint` |
| Jump | `Space` | `IA_Jump` |
| Crouch | `Left Ctrl` | `IA_Crouch` |
| Interact | `E` | `IA_Interact` |
| Grab and carry (hold, release to drop) | `Left Mouse` | `IA_Grab` |
| Use the selected item, fire a gun | `Right Mouse` | `IA_UseItem` |
| Reload | `R` | `IA_Reload` |
| Drop the selected item | `G` | `IA_DropItem` |
| Select an inventory slot | `1` `2` `3` `4` | `IA_SelectSlot` |
| Step through the slots, or push and pull a carried object | mouse wheel | `IA_ScrollSlot` |
| Ping | `Middle Mouse` | `IA_Ping` |
| Danger ping | `Middle Mouse` twice, quickly | `IA_PingDanger` |
| Emote wheel (hold, release to pick) | `T` | `IA_Emote` |
| Pause | `P` | `IA_PauseMenu` |

While the emote wheel is open, the mouse wheel turns its pages (`IA_EmoteWheelPage` in `IMC_EmoteWheel`). While you spectate, `W` `A` `S` `D` fly the free camera, `Space` goes up and `Left Ctrl` goes down (`IA_SpectateMove` in `IMC_Spectator`). Those six keys are six rows of the CONTROLS list, in the `Spectator` group.

A melee hit has no key of its own. Grab an object that carries a melee component and swing the mouse. See [Turn any grabbable object into a melee weapon](../weapons/make_a_melee_weapon.md).

Keyboard and mouse only: no context ships a gamepad mapping.

---

## Change a key

1. Open `Content/MPFriendslop/Inputs/IMC_Default`.
2. Find the row for the action.
3. Click the key field and press the new key.
4. Save.

`BP_FriendslopPlayerController` listens to the action, not to the key, so nothing else needs an edit.

Do not delete a row and add a new one when the row carries modifiers. Change the key on the existing row instead:

- `IA_Move` needs its `Negate` and `Swizzle Input Axis Values` modifiers to turn four keys into one direction. Each of its four rows also carries its own rebind name, so the CONTROLS list shows one row per direction.
- Each `IA_SelectSlot` row carries the `Scalar` that tells it which slot it is, and its own rebind name (`SelectSlot1` to `SelectSlot4`), so the CONTROLS list shows one row per slot.

Players rebind keys in game from the **CONTROLS** tab of the settings. That is covered in [Settings and volume sliders](../ui/settings_and_audio.md).

---

## Add a new input

1. Right click in `Inputs/Inputs/`, then **Input**, then **Input Action**. Name it `IA_<Something>` and set its `Value Type`.
2. Open `IMC_Default`, add a mapping, pick your action and press its key.
3. In `BP_FriendslopPlayerController`, add the Enhanced Input event for your action and wire what it does. To drive a component on the pawn, copy what the controller does for the shipped actions: `Get Controlled Pawn`, `Get Component by Class`, `Is Valid`, then call the component.
4. To make it rebindable in game, open the action and fill `Player Mappable Key Settings`: a `Name`, a `Display Name` and a `Display Category`.

| Field on the action | What it does |
|---|---|
| `Name` | The key the player's saved binding is stored under. An action without it never shows in the CONTROLS list. |
| `Display Name` | The row label in the CONTROLS list. |
| `Display Category` | The group the row sits in. The shipped groups are `Movement`, `Interaction`, `Communication`, `Interface` and `Spectator`. |

When one action has several keys that must each be rebound on their own, like the four directions of `IA_Move`, the fields go on each row of the mapping context instead of on the action. Open the `IMC_`, expand the row, set `Setting Behavior` to `Override Settings`, and fill the same three fields there, with a different `Name` per row. `IA_Move`, `IA_SelectSlot` and `IA_SpectateMove` are set up this way.

The mouse look (`IA_Look`) and the two mouse wheel actions (`IA_ScrollSlot` and `IA_EmoteWheelPage`) have no `Player Mappable Key Settings`, so none of them shows in the CONTROLS list. An axis rebound on a single key could only step one way.

---

## Add your own mapping context

For a mode that needs its own keys (a vehicle, a camera, a minigame), make a new `IMC_`. Add it with **Add Mapping Context** on the Enhanced Input local player subsystem, at a priority above `0` so it wins over `IMC_Default`. Remove it when the mode ends. The emote wheel does exactly this, with its `Wheel Context` and `Wheel Context Priority` fields set to `IMC_EmoteWheel` and `1`.

For its actions to be rebindable, the CONTROLS list must know the context. Open `WBP_SettingsPage`, select the placed `WBP_ControlsPanel`, and add your context to its `Mapping Contexts` array next to `IMC_Default`, `IMC_EmoteWheel` and `IMC_Spectator`.

!!! warning
    If you replace `Default Context` on the controller with a context of your own, it must map the same `IA_` assets. The template's components listen for those exact actions, so a context that maps other actions to the same keys leaves sprint, grab, the inventory and the rest silent, with no error.

---

## Key icons

The CONTROLS list draws each key from `Blueprints/DataAssets/KeyIcon/Childs/DA_KeyIcon_Keyboard`, a `BP_KeyIconDataAsset` with two fields.

| Field | What it does |
|---|---|
| `Icons` | One row per key: the `Key` and its `Icon` texture. |
| `Fallback Icon` | Drawn for a key with no row. Left empty, so such a key shows its engine name as text instead. |

A key you add that has no row still works, it just shows as text until you add one.

For a prompt of your own, `Blueprints/Functions/BPFL_InputPrompt` has `Get Input Action Prompt`. Give it the player controller, an Input Action and the key icon asset, and it returns the key the player has bound right now, its icon, and the action's display name and category.

---

Next: [How the player character works](../player/how_the_player_works.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
