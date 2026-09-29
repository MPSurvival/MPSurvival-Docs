# Death, revive and the spectator

A player who dies loses their head. The head drops to the floor, a countdown starts, and the others have until the end of it to carry the head to a revive bay. If nobody does, the dead player becomes a free camera until the next run.

Two components own all of it:

- `BP_DeathComponent` on the character: the death, the head, the countdown and the revive.
- `BP_SpectatorComponent` on the player controller: the free camera.

Both are in `Content/MPFriendslop/Blueprints/ActorComponents/`.

---

## What happens, in order

| Moment | What the players see |
|---|---|
| Death | The body disappears and a `BP_PlayerHead` pops off the `head` bone with the dead face. The player lets go of what they held, and their inventory drops around the body. There is no ragdoll |
| Bleedout | The dead player looks through the head's camera, in black and white, with a large countdown. They can no longer move or turn the view, but the pause key still works. The others see a skull and the countdown in red above the head, through walls |
| Revive | Someone carries the head into a `BP_ReviveBay` and holds `E`. The player is rebuilt standing in front of the bay, with `Revive Health` |
| Bleedout ends | Head and body are destroyed. The dead player becomes a spectator |
| Next run | Spectators are brought back as normal players |

The head is grabbed like loot, but it is not loot: it cannot be sold. When every player is down, the crew falls into the Last Loser Standing arena, or the run ends when there is none. See [How a run works](../loot/how_a_run_works.md).

A player killed in the arena skips the countdown and becomes a spectator a moment later. See [The Last Loser Standing arena](../loot/the_arena.md).

A player who jumps into an open extraction hatch also becomes a spectator, and so does a player who joins while the end of run recap is open.

---

## The death fields

On `BP_DeathComponent`, in `Settings|Death`. Change them on the component inside `BP_FriendslopCharacter`: that copy is the one that runs.

| Field | What it does | Shipped on the character |
|---|---|---|
| `Head Class` | The actor spawned at the `head` bone. Make a child of `BP_PlayerHead` and point this at it | `BP_PlayerHead` |
| `Bleedout Seconds` | Length of the countdown, so also the time the others have to revive | `60` |
| `Death Screen Class` | The full screen widget shown to the dead player only | `WBP_DeathScreen` |
| `Decapitation Speed` | Upward speed given to the head when it spawns, in cm/s | `250` |
| `Revive Health` | Health given back when the player is rebuilt at a bay | `50` |

`WBP_DeathScreen` is created by this component, not by the HUD. The skull marker above the head is `WBP_DeadHead`. Neither has settings: restyle them in the designer. How heavy and how bouncy the head feels is on its `Mesh`, in the Details panel of `BP_PlayerHead`.

To kill a player from your own trap or volume, call `Kill` on their `BP_DeathComponent` from server code. `BP_DebugKillVolume` does exactly that.

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnDecapitated` | The player died and the head spawned | server |
| `OnBleedoutExpired` | The countdown ended, or the run ended during it | server |
| `OnRevived` | The player was rebuilt at a bay | server |

---

## The spectator

The spectator is a free flying camera: `W A S D` to move, `Space` up, `Ctrl` down. There is no mode that follows a living player.

The camera is stopped by walls, floors and ceilings, so it stays inside the level. It passes through doors, players, enemies and loot. This is the engine's `Spectator` collision preset on `BP_SpectatorPawn`: it blocks `WorldStatic` only, so any wall you place as a static mesh keeps the spectator in.

`BP_SpectatorComponent` is on `BP_FriendslopPlayerController`, because a dead player has no character left to carry it.

| Field | What it does | Shipped default |
|---|---|---|
| `Spectator Context` | Input mapping context added while spectating | `IMC_Spectator` |
| `Spectator Context Priority` | Its priority. Above `IMC_Default` (`0`) so the movement keys fly the camera | `1` |
| `Spectator Widget Class` | The overlay: `SPECTATING` and one hint line | `WBP_Spectator` |
| `Spectator Pawn Class` | The invisible pawn the player flies | `BP_SpectatorPawn` |
| `Camera Modifier Class` | The post process on the spectator's view | `BP_SpectatorCameraModifier` |

The desaturated look is `Spectator Post Process` on `BP_SpectatorCameraModifier`. It is a full post process block: add grain, a vignette or a colour grade there, no graph needed.

`OnSpectatorStateChanged` gives `Is Spectating`. It fires on the server and on the owning player. The template's player controller uses it to hide the gameplay HUD. Bind it the same way to hide your own. `WBP_Spectator` comes with the component, not with the HUD.

---

## On your own character

1. Add `BP_DeathComponent` to your pawn.
2. Implement `BPI_VitalManagerInterface` on the pawn and return a `BP_VitalsSystem` from `GetVitalComponent`. That is where the death comes from. See [Health, damage and new vitals](../player/health_and_damage.md).
3. Give the pawn a skeletal mesh with a bone or socket named `head`. The name is fixed.
4. Put `BP_SpectatorComponent` on your player controller, and implement `BPI_PlayerLifeState` on your player state so the run knows who is down.
5. To have the end of the run and the start of the arena cut a bleedout short, forward `NotifyRunEnded` and `NotifyArenaStarted` from `BPI_RunControl` to `EndBleedoutNow`, as `BP_FriendslopCharacter` does.

Steps 4 and 5 are optional. Without them, that part simply does nothing. The full list of components is in [How the player character works](../player/how_the_player_works.md#your-own-character-blueprint).

---

Next: [Place a revive bay](place_a_revive_bay.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
