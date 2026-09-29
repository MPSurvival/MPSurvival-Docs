# Use your own game framework classes

You can swap the Game Instance, Game Mode, Game State, Player State or Player Controller for your own and keep every system of the template working. Nothing in the template casts to these classes. The Game Modes name them as class defaults, and every other Blueprint reaches them through an interface message. So your class only has to implement the right interfaces.

If you only want your own pawn, you do not need this page. See [How the player character works](../player/how_the_player_works.md#put-it-on-your-own-character).

- The classes: `Content/MPFriendslop/Blueprints/Misc/` and `Blueprints/PlayerCharacter/`
- The interfaces: `Blueprints/Interfaces/`

---

## Where each class is set

| Class | Set in | Shipped |
|---|---|---|
| Game Instance | **Project Settings**, **Maps & Modes**, `Game Instance Class` | `BP_FriendslopGameInstance` |
| Game Mode of the run | **Project Settings**, **Maps & Modes**, `Default GameMode` | `BP_FriendslopGameMode` |
| Game Mode of the menu | World Settings of `L_MainMenu`, `GameMode Override` | `BP_MainMenuGameMode` |
| Game State, Player State, Player Controller | `Game State Class`, `Player State Class`, `Player Controller Class` on each Game Mode | See the table below |

| On the Game Mode | `BP_FriendslopGameMode` | `BP_MainMenuGameMode` |
|---|---|---|
| `Game State Class` | `BP_FriendslopGameState` | `BP_LobbyGameState` |
| `Player State Class` | `BP_FriendslopPlayerState` | `BP_FriendslopPlayerState` |
| `Player Controller Class` | `BP_FriendslopPlayerController` | the engine `PlayerController` |

Both Game Modes are `Game Mode Base`, so a Game State of your own must derive from `Game State Base`, not from `Game State`.

---

## Pick how to start

| Start from | Take this one when | What it brings |
|---|---|---|
| A child of our class | You want to add to it | Everything keeps working. Add your variables and events, then set the child in the fields above |
| Your own class | It already exists in your project, with another parent | You implement the interfaces below yourself. A function you leave empty turns that feature off, with no error |

A child is the short road for the Game Mode and the Player Controller. The Game Mode holds the run rules. The Player Controller holds the input, the HUD and the pause menu.

---

## What each class must answer

| Class | Implement | Also needs | Replicated state it carries |
|---|---|---|---|
| Game Instance | `BPI_Session`, `BPI_GameSettings`, `BPI_PlayerProfile` | The `Settings\|Session`, `Settings\|Profile`, `Settings\|Cosmetics` and `Settings\|Audio` fields of `BP_FriendslopGameInstance`, and its `NotifySettingsListeners`, called at the end of `SetGameSettings` | None. It is local to each machine |
| Game State of the run | `BPI_ExtractionBank`, `BPI_RunState` | The `Settings\|Arena` fields `Kill Reward` and `Win Reward` | The quota, the extracted value, the run end time, whether the run has ended, whether a player made it out, the value left behind, the manifest of the run, whether the arena took place and who won it |
| Game State of the menu | `BPI_Session` | Pass the calls on to the Game Instance | The host options |
| Player State | `BPI_LobbyMember`, `BPI_PlayerColor`, `BPI_PlayerLifeState`, `BPI_ArenaScore`, and `RequestRestart` of `BPI_RunControl` | `BP_PlayerColorComponent` and `BP_CosmeticComponent` | The lobby status, whether the player is down, whether the player is out, the player's kills in the arena |
| Game Mode of the run | `RestartRun`, `NotifyPlayerDownChanged`, `NotifyPlayerEvacuated` and `NotifyArenaFinished` of `BPI_RunControl` | Set the quota with `SetQuota` and start the run with `BeginRun` on the Game State. Wait until the level is ready and every player has arrived, as `TryStartRun` does. When the crew is down, start the arena as `TryStartArena` does | None. It exists on the server only |
| Player Controller | `EvacuatePlayer`, `NotifyRunEnded` and `NotifyArenaStarted` of `BPI_RunControl`, and `BPI_GameSettingsListener` | `BP_SpectatorComponent`, and the pieces listed in the next section | None of its own |

The Player State is used in both maps: the lobby reads each player's status and colour from it before the run starts. Death sets `SetIsDown` on it, and that is how the Game Mode knows the whole crew is down. How the lobby uses `BPI_LobbyMember` is on [Sessions and the lobby](sessions_and_the_lobby.md).

---

## Your own Player Controller

No interface covers the controller's own job. It adds the input, creates the HUD and opens the pause menu. Copy these from `BP_FriendslopPlayerController`:

| Field | Category | Shipped value |
|---|---|---|
| `Default Context` | `Settings\|Input` | `IMC_Default` |
| `Gameplay Widget Class` | `Settings\|HUD` | `WBP_Gameplay` |
| `Recap Widget Class` | `Settings\|HUD` | `WBP_RunRecap` |
| `Pause Widget Class` | `Settings\|HUD` | `WBP_MenuRoot` |
| `Pause Page Class` | `Settings\|HUD` | `WBP_PausePage` |
| `Arena Intro Widget Class` | `Settings\|HUD` | `WBP_ArenaIntro` |
| `Loading Widget Class` | `Settings\|HUD` | `WBP_LoadingScreen` |

Then copy the functions that use them: `ShowGameplayWidget`, `ShowRecapWidget`, `ShowLoadingWidget`, `RefreshHud`, `OpenPauseMenu`, `ClosePauseMenu`, `PrepareRouter`, `RefreshSettings`, `Move`, `Look`, `SetJumpInput`, `SetCrouchInput` and `EvacuateOwnPawn`, and the input events of `IA_Move`, `IA_Look`, `IA_Jump`, `IA_Crouch` and `IA_PauseMenu`. Copy the twelve other input events too, from `IA_Interact` to `IA_EmoteWheelPage`: they pass each key to its component on the pawn, and without them interaction, sprint, grab, the inventory, pings and emotes get no input. The list is on [How the player character works](../player/how_the_player_works.md). `RefreshSettings` applies the mouse sensitivity and invert Y it is given: `BeginPlay` passes it `GetGameSettings`, and the `ApplyGameSettings` event passes it every later change. What each HUD piece needs is on [The HUD, and adding to it](the_hud.md).

`EvacuatePlayer` is the one people forget. Once the quota is met, the extraction point sends it to the controller of each player standing in it. A controller that does not answer it never leaves the map. `NotifyRunEnded` is how each player gets paid: it adds the crew share, plus what the player earned in the arena, to the credits of that player's Game Instance. `NotifyArenaStarted` sends `Client_PlayArenaIntro` to the owning player, which shows `Arena Intro Widget Class`. See [The Last Loser Standing arena](../loot/the_arena.md).

---

For the order the Game Mode runs things in, see [How a run works](../loot/how_a_run_works.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
