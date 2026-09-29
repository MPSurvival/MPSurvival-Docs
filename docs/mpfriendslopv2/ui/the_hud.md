# The HUD, and adding to it

The in-run HUD is one widget, `WBP_Gameplay`, created for each player by `BP_FriendslopPlayerController`. Each piece finds what it shows by itself, on the pawn or on the game state. So a widget of your own drops in without touching the character.

- The HUD widgets: `Content/MPFriendslop/Blueprints/Widgets/Gameplay/`
- The controller: `Content/MPFriendslop/Blueprints/PlayerCharacter/BP_FriendslopPlayerController`

---

## What is in it

`WBP_Gameplay` has two layers. Most pieces sit in `ScreenContent`, inside a `RetainerBox` called `Screen`. Its material, `MI_Widget_HUDScreen`, bends the HUD into a curved screen. Three pieces sit outside it, flat, directly under `Root`.

| Piece | Layer | What it shows | It reads |
|---|---|---|---|
| `WBP_Vitals` | curved | Health and stamina, top left | the vitals component, through `BPI_VitalManagerInterface` |
| `WBP_Crosshair` | curved | The dot in the middle | `BP_EmoteComponent`, to hide its dot while the emote wheel is open |
| `WBP_Interact` | curved | The hold ring and the prompt line | `BP_InteractionComponent` |
| `WBP_Inventory` | curved | The quick slots, bottom right | `BP_InventoryComponent` |
| `WBP_RunClock` | curved | Time left in the run | nothing: `WBP_Gameplay` feeds it from the game state, through `BPI_RunState` |
| `WBP_ExtractionTotals` | curved | Extracted value against the quota | the game state, through `BPI_ExtractionBank` |
| `WBP_ObjectiveTitle` | curved | The title shown once the quota is met | the game state, through `BPI_ExtractionBank` |
| `WBP_GrabHand` | flat | The open and closed hand on a grabbable | `BP_GrabComponent` |
| `WBP_PingLayer` | flat | Pings, and arrows at the screen edge for pings out of view | the ping actors in the level |
| `WBP_DamageVignette` | flat | The red flash when you get hurt | `BP_DamageFeedbackComponent` |

A piece whose component is missing from the pawn stays idle. There is no error, so a pawn without an inventory simply shows no slots.

Two rules hide the crosshair, and each one sets a different widget:

- The Tick of `WBP_Gameplay` sets the visibility of its `Crosshair` widget every frame: hidden while the grab hand is visible, shown otherwise. To hide that widget for another reason, do it in that same Tick, or it is overwritten on the next frame.
- Inside, `WBP_Crosshair` hides its own `CrosshairDot` while the emote wheel is open. It listens to `OnWheelToggled` on the pawn's `BP_EmoteComponent`.

Since they never set the same widget, neither one undoes the other.

| Field | Where | What it does | Shipped default |
|---|---|---|---|
| `Gameplay Widget Class` | `BP_FriendslopPlayerController` | The HUD created for the player. Empty means no HUD | `WBP_Gameplay` |
| `Run State Interval` | `WBP_Gameplay` | How often, in seconds, the run clock is refreshed | `0.25` |

The fields of each piece are on the page of its system: [vitals](../player/health_and_damage.md), [interaction](../interaction/how_interaction_works.md), [grabbing](../grab/how_grabbing_works.md), [the inventory](../items/how_items_work.md), [the run clock and totals](../loot/tune_the_run.md), [pings](../crew/colours_nameplates_and_pings.md).

---

## When the HUD hides

The controller's `RefreshHud` function is the only thing that hides the HUD. It collapses `ScreenContent` while the pause menu or the end-of-run recap is open, and while the player is spectating. The flat pieces under `Root` stay on screen.

During the [Last Loser Standing arena](../loot/the_arena.md), `WBP_Gameplay` itself hides the run clock, the extraction totals and the objective title, in its `ApplyArenaLayout` function. Health, stamina and the quick slots stay.

Nothing in the HUD is clickable. The curve moves what you see but not where the mouse lands, so a button inside `Screen` would never be hit where it is drawn.

---

## Add a widget to the HUD

1. Create a Widget Blueprint in `Content/MPFriendslop/Blueprints/Widgets/Gameplay/`.
2. In **Event Construct**, bind `On Possessed Pawn Changed` on `Get Owning Player` to a custom event.
3. Still in **Event Construct**, call your own attach function with `Get Owning Player Pawn`.
4. From the custom event, call the same attach function with `New Pawn`.
5. In the attach function, look up what you need on the pawn with `Get Component By Class` or an interface call. If it is valid, call `Unbind Event from` with your own event, then `Bind Event to` with the same event. If not, do nothing.
6. Open `WBP_Gameplay` and place your widget under `ScreenContent` to have it curved and hidden with the HUD, or under `Root` to keep it flat and always on screen.

Steps 2 to 4 are what `WBP_Vitals` and `WBP_DamageVignette` do. They matter because the HUD is often created before the pawn exists. The controller also changes pawn when the player becomes a spectator, and again when they come back. A widget that looks at the pawn only once, on construct, stays empty on a client, or keeps pointing at the wrong pawn afterwards.

Step 5 unbinds your own event only. Never use `Unbind all Events from` on a pawn's dispatcher: other listeners share it, such as `BP_DamageFeedbackComponent` on `OnVitalsChanged`, and they would stop firing with no error.

---

## Show a new vital

`WBP_Vitals` draws one row per vital, placed by hand: `HealthRow` and `StaminaRow`, inside a Vertical Box called `Rows`. A vital you add to `BP_VitalsSystem` needs a row of its own, and that is an edit of `WBP_Vitals`. Here the new vital is called Hunger.

1. Open `WBP_Vitals`. In the **Hierarchy**, right click `StaminaRow`, then **Duplicate**.
2. Rename the copy `HungerRow`, and its three children `HungerIcon`, `HungerValue` and `HungerMax`. Check that `Is Variable` is ticked on all three.
3. Select `HungerIcon` and set `Brush`, then `Image`, to your icon texture. The shipped icons are in `Content/MPFriendslop/Textures/Widgets/Vitals/`.
4. Open the `AttachToPawn` function. It calls `ApplyVitalStyle` once per row. Add a third call. Feed its `Vital` input with `GetVital` on the same vitals component, set to your vital type, and its three other inputs with `HungerIcon`, `HungerValue` and `HungerMax`.
5. Open the `RefreshValues` function. It calls `RefreshVital` once per row. Add a third call with `GetVital` set to your vital type, and `HungerValue`.

`ApplyVitalStyle` runs once, when the pawn is found. It tints the icon and both numbers with the `Vital Color` of the vital's Data Asset, and writes the small max text, such as `/100`, from its `Vital Max Amount`. `RefreshValues` runs each time `OnVitalsChanged` fires, and writes the current amount, rounded down.

---

## Replace the whole HUD

Point `Gameplay Widget Class` on the controller at your own widget. The shipped pieces need nothing from `WBP_Gameplay`, so place the ones you want to keep in yours, `WBP_Inventory` included.

Two things come from `WBP_Gameplay` itself, and you have to redo them:

- feeding the run clock: call `SetRemainingSeconds` on `WBP_RunClock`, with the value from `BPI_RunState`
- folding the crosshair while the grab hand is visible

!!! warning
    `RefreshHud` finds the part to hide by casting the HUD to `WBP_Gameplay` and reading `ScreenContent`. A HUD of any other class is never hidden during pause, recap or spectating, with no error. Either edit `WBP_Gameplay` in place, or change that cast in `RefreshHud` to your class.

---

## What does not come from the HUD

These screens are created by their own component or field, not by `Gameplay Widget Class`. Replacing the HUD leaves them working.

| Widget | Created by | Field |
|---|---|---|
| `WBP_DeathScreen` | `BP_DeathComponent` | `Death Screen Class` |
| `WBP_Spectator` | `BP_SpectatorComponent` | `Spectator Widget Class` |
| `WBP_EmoteWheel` | `BP_EmoteComponent` | `Emote Wheel Class` |
| `WBP_RunRecap` | `BP_FriendslopPlayerController` | `Recap Widget Class` |
| `WBP_LoadingScreen` | `BP_FriendslopPlayerController` | `Loading Widget Class` |

`WBP_LoadingScreen` removes itself: see [How procedural levels work](../world/how_procedural_levels_work.md). The walking robot is one texture played by the material `MI_Widget_LoadingRobot`. To tint it, change `Color and Opacity` on its `LoadingRobot` image.

To hide your own HUD while spectating, bind `OnSpectatorStateChanged` on `BP_SpectatorComponent`, as the controller does.

---

Next: [Settings and volume sliders](settings_and_audio.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
