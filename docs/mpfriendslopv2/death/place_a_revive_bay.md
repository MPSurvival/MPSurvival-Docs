# Place a revive bay

A dead player can come back while their head still exists. A teammate carries the head to a revive bay, drops it in the cradle, and holds the interact key. The player is rebuilt standing in front of the machine. By the end of this page you have a bay in your level, and you know how to make your own remains work with it.

- The bay: `Content/MPFriendslop/Blueprints/Environments/Fixtures/BP_ReviveBay`
- Its sign: `Blueprints/Widgets/World/WBP_ReviveSign`
- The two interfaces it talks through: `Blueprints/Interfaces/BPI_PlayerRemains` and `BPI_Revivable`

---

## How the bay decides

The server scans the bay's `CradleZone` every `Detection Interval`. When it finds a head whose owner is still down, it snaps the head into `HeadSlot` and the bay becomes `Loaded`. The bay's status is an `E_ReviveStatus`, and the sign shows it.

| Status | When | Sign |
|---|---|---|
| `Empty` | No head in the cradle | `Bring a head here` |
| `Loaded` | A head is snapped in. A player can now hold interact | `Hold to rebuild` |
| `Working` | The player was just rebuilt. The clamps and the scanner play their animation for `Cycle Duration` | `REBUILDING...` |

The player is rebuilt the moment the hold completes, at the start of `Working`. The cycle that follows is only the animation, then the bay goes back to `Empty`.

A revive only works during the bleedout. When `Bleedout Seconds` runs out, the head is destroyed and there is nothing left to bring back. That timer and the health given back (`Revive Health`) live on the player's `BP_DeathComponent`. See [Death, revive and the spectator](how_death_works.md).

---

## Place the bay

1. Drag `BP_ReviveBay` into the level.
2. Leave free floor in front of it. The revived player appears at the `ExitPoint` component.
3. If you move the bay against a wall or change its shape, select `ExitPoint`, `HeadSlot` or `CradleZone` in the Components panel and move them to match.

The bay is ready. It needs no link to a door, a button or a spawn point.

| Component | What it does |
|---|---|
| `CradleZone` | The box that finds a head dropped in the bay |
| `HeadSlot` | Where the head is held while the bay is `Loaded` |
| `ExitPoint` | Where the revived player appears |
| `Display` | Draws `WBP_ReviveSign` |
| `JawLeft`, `JawRight`, `Gantry` | The clamps and the scanner that move during `Working` |

---

## The fields

All of them are on the placed `BP_ReviveBay`.

| Field | Category | What it does | Default |
|---|---|---|---|
| `Prompt` | `Settings|Interaction` | The text of the interaction prompt | `Rebuild` |
| `Type` | `Settings|Interaction` | How the player interacts | `Hold` |
| `Duration` | `Settings|Interaction` | Seconds to hold | `1.5` |
| `Detection Interval` | `Settings|Revive` | Seconds between two scans of `CradleZone` | `0.2` |
| `Detection Object Types` | `Settings|Revive` | The collision object types the scan looks for | `WorldDynamic`, `Pawn`, `PhysicsBody` |
| `Cycle Duration` | `Settings|Revive` | Seconds of the clamp and scanner animation after a rebuild | `5` |
| `Jaw Stroke` | `Settings|Revive` | How far each clamp moves, in cm | `8` |
| `Gantry Drop` | `Settings|Revive` | How far the scanner moves, in cm | `-10` |

The sign's texts and colours are on `WBP_ReviveSign`: `Empty Text`, `Loaded Text`, `Working Text`, and `Empty Color`, `Loaded Color`, `Working Color`. Change them in the widget to change every bay.

---

## Your own remains

The bay does not know the head, the character or the death component. It reaches all three through interfaces, so any of them can be yours.

1. On the actor that stands for a dead player (a corpse, a dog tag, a soul), implement `BPI_PlayerRemains`:
    - `Get Remains Owner` returns the dead player's PlayerState.
    - `Snap Remains To` receives the `HeadSlot` location and rotation. Move your actor there, stop its physics, and return `Snapped` true. Return false to refuse, for example while someone holds it.
2. Give that actor a collision object type listed in the bay's `Detection Object Types`.
3. Make sure the owner's PlayerState answers `Get Is Down` from `BPI_PlayerLifeState` with true while the player is dead.
4. On a **component** of the dead player's pawn, implement `BPI_Revivable`. `Revive At` receives the `ExitPoint` location and rotation: bring the player back there and remove your remains.

`BP_PlayerHead` and `BP_DeathComponent` do exactly this. Open them for a working example.

!!! warning
    The bay looks for `BPI_Revivable` on the pawn's components only. Implemented on the pawn itself, it is never found: the hold completes, the bay plays its cycle, and nobody comes back. There is no error.

---

## The dispatchers

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnStatusChanged` | The bay's status changed. Gives the new `Status` | every player |
| `OnPlayerRebuilt` | A player was rebuilt. Gives their PlayerState | server |

---

Next: [How cosmetics work](../crew/how_cosmetics_work.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
