# Settings and volume sliders

The `SETTINGS` screen has four tabs: `GAME`, `GRAPHICS`, `AUDIO` and `CONTROLS`. There is no `APPLY` button: a `< VALUE >` selector applies as soon as it changes, a slider when you let go of it. This page shows where each value is kept, how to add a setting of your own, and how to put your own sounds under the volume sliders.

- The screen and its rows: `Content/MPFriendslop/Blueprints/Widgets/Menu/`
- The settings struct: `Content/MPFriendslop/Blueprints/Structures/S_GameSettings`
- The sound classes and the mix: `Content/MPFriendslop/Audios/`

The same `WBP_SettingsPage` opens from the main menu and from the pause menu.

---

## Where each value lives

| Tab | Rows | Stored in |
|---|---|---|
| `GAME` | `MOUSE SENSITIVITY`, `INVERT LOOK Y`, `FIELD OF VIEW` | `S_GameSettings`, on the Game Instance, saved in the profile |
| `GRAPHICS` | `WINDOW MODE`, `RESOLUTION`, `VSYNC`, `MAX FRAMERATE`, `RENDER SCALE`, `SHADOWS`, `TEXTURES`, `EFFECTS`, `VIEW DISTANCE` | The engine's `GameUserSettings`, in `GameUserSettings.ini`. Never copied into the profile |
| `AUDIO` | `MASTER VOLUME`, `SOUND EFFECTS`, `MUSIC`, `INTERFACE` | `S_GameSettings`, on the Game Instance, saved in the profile |
| `CONTROLS` | the rebind list | The engine's Enhanced Input user settings. See [The controls, and adding an input](../start/controls.md) |

`S_GameSettings` holds `MouseSensitivity` (1), `InvertLookY` (off), `FieldOfView` (90) and the four volumes (1 each). `BP_FriendslopGameInstance` keeps it. It saves it in the one profile save, `SG_FriendslopProfile` (slot `FriendslopProfile`), together with the credits and the cosmetics. To read it from another Blueprint, call `GetGameSettings` on the Game Instance, through `BPI_GameSettings`.

A change applies at once, even in the middle of a run. Each time a setting changes, the Game Instance calls `ApplyGameSettings` from `BPI_GameSettingsListener` on the local player controller, on its pawn, and on every component of that pawn that implements it. `BP_FriendslopPlayerController` uses it for the look settings, and `BP_ViewMotionComponent` for the field of view. Both also read `GetGameSettings` once in their `BeginPlay`, for the values saved before they existed.

The volume sliders run from 0 to 100 on screen and are stored from 0 to 1.

---

## The two row widgets

Every row on the screen is one of these two widgets, placed in `WBP_SettingsPage`.

`WBP_SettingsSlider`:

| Field | What it does | Default |
|---|---|---|
| `Label` | The text on the left | `SETTING` |
| `Min Value` / `Max Value` | The range of the slider | 0 / 100 |
| `Decimals` | Decimals shown next to the slider | 0 |
| `Tick Sound` | Played while you drag | `CUE_UIClick` |
| `Tick Notches` | How many notches along the slider play `Tick Sound` | 20 |

`WBP_SettingsCombo`, the `< VALUE >` selector:

| Field | What it does |
|---|---|
| `Label` | The text on the left |
| `Options` | The values it steps through, as `Text` |
| `Hover Sound` / `Click Sound` | `CUE_UIHover` / `CUE_UIClick` |

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnSliderCommitted` | The player lets go of the slider. Gives `Value` | owning player |
| `OnComboSelected` | The player steps to another option. Gives `Index` | owning player |

Setting a row's value from a graph (`SetValue`, `SelectIndex`) does not fire its dispatcher, so filling the screen when it opens does not save anything.

---

## Add a setting

This is an edit of `WBP_SettingsPage` and `S_GameSettings`. Copy an existing row all the way: `MOUSE SENSITIVITY` for a slider, `INVERT LOOK Y` for a selector.

1. Open `S_GameSettings` and add your field, with its default value.
2. Open `WBP_SettingsPage`. Place a `WBP_SettingsSlider` or a `WBP_SettingsCombo` next to the other rows of the tab you want, and tick `Is Variable`.
3. Fill its `Label`, then `Min Value`, `Max Value` and `Decimals`, or `Options`.
4. In its Details, add the `OnSliderCommitted` or `OnComboSelected` event, and call a new handler function from it.
5. In the handler, break `Current Settings`, make a new `S_GameSettings` with your value on your field and every other pin wired from the break, set `Current Settings`, then call `PushGame`. `HandleSensitivity` is exactly this.
6. In `LoadFromGame`, give your row its value from `Current Settings` with `SetValue` or `SelectIndex`, so the screen opens on the saved value.
7. In the Blueprint that uses the setting, call `GetGameSettings` on the Game Instance in `BeginPlay` and read your field. To follow later changes, add `BPI_GameSettingsListener` to that Blueprint (the controller, the pawn, or a component on the pawn) and read the same field in its `ApplyGameSettings` event.

!!! warning
    Adding a field to `S_GameSettings` adds a pin to every `Make S_GameSettings` node in the other handlers of `WBP_SettingsPage`. Wire that pin from the `Break` in each one. A pin left unwired writes the default value, so your setting silently goes back to its default every time the player changes any other setting.

A profile saved before your field existed reads it as 0. If 0 is not a valid value for your setting, treat 0 as "use the default", the way `RefreshSettings` falls back to a sensitivity of 1.

For a graphics setting, skip `S_GameSettings`. In the handler, call `Get Game User Settings`, set your value on it, then call `ApplyUser`, which applies and saves it. The existing `HandleShadows` is the model.

---

## Put your sounds under the volume sliders

The sliders drive four Sound Classes through the mix `SMIX_UserAudio`:

| Slider | Sound Class |
|---|---|
| `MASTER VOLUME` | `SCLS_Master` |
| `SOUND EFFECTS` | `SCLS_Effects` |
| `MUSIC` | `SCLS_Music` |
| `INTERFACE` | `SCLS_UI` |

`SCLS_Effects`, `SCLS_Music` and `SCLS_UI` are children of `SCLS_Master`, so the two volumes multiply: music at 50 with master at 50 plays at a quarter. `SCLS_Dialogue` and `SCLS_Cinematics` are also under `SCLS_Master` and follow only the master slider.

To hook up a sound:

1. Open your Sound Cue, Sound Wave or MetaSound.
2. In the Details panel, under `Sound`, set `Sound Class` to one of the classes above.

A sound with no `Sound Class` does not follow any of the sliders, not even `MASTER VOLUME`. Check this first when a sound stays loud at volume 0.

To point the sliders at your own classes, change the five fields in `Settings|Audio` on `BP_FriendslopGameInstance`: `Audio Mix`, `Master Class`, `Sfx Class`, `Music Class` and `Ui Class`. A fifth slider for a new class needs three things: the recipe above, a Sound Class variable on the Game Instance, and one more `Set Sound Mix Class Override` in its `ApplyAudio` function.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
