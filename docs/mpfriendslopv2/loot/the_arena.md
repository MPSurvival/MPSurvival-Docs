# The Last Loser Standing arena

When the whole crew is down, the run does not end at once. The screen goes black, and every player falls onto a round platform in the dark, with weapons on the floor. The last one standing breaks a glass case and takes the crown. Kills and the win pay credits, and the recap gets a `LAST LOSER STANDING` block.

The arena is one actor, `BP_Arena`. The GameMode only looks for an actor that implements `BPI_Arena`, so removing the arena or replacing it with your own needs no change to the GameMode. If you have not read [How a run works](how_a_run_works.md), start there.

- The arena, the glass case, the crown and the giant head: `Content/MPFriendslop/Blueprints/Environments/Arena/`
- The black screen: `Blueprints/Widgets/Gameplay/WBP_ArenaIntro`
- The head's screen: `Blueprints/Widgets/World/WBP_ArenaMessage`
- The interface: `Blueprints/Interfaces/BPI_Arena`

---

## What happens, in order

| Phase | What the players see |
|---|---|
| `Blackout` | Every screen goes black and flickers, with a power-on sound, for `Blackout Seconds`. The weapons appear on the platform |
| `Falling` | Every player is brought back standing, high above the platform, and falls onto it. The giant head over the platform says `LAST LOSER STANDING`. Nobody can be hurt for `Spawn Protection Seconds`, the fall included |
| `Fighting` | The players fight with what they find. After `Ring Drop Interval` seconds of fighting, the outer ring flashes red, the head says `THE FLOOR IS LEAVING` and an alarm comes out of its mouth. `Ring Warning Seconds` later the ring falls. The next warning comes `Ring Drop Interval` seconds after that: three rings, then the core |
| `LastStanding` | One player is left. The floor stops falling, the glass case unlocks and the head says `BREAK THE GLASS`. The survivor breaks the glass with a hit or a shot, then takes the crown with the interact key |
| `Over` | The crown floats above the winner's head and the head says `NAME WINS`. `Win Celebration Seconds` later, the run ends and the recap opens |

If the last players die together, a fall of the core included, nobody wins: the head says `NOBODY WINS` and the run ends `Win Celebration Seconds` later. When the core falls, the glass case, the crown, the head and the lights stay in the air. The glass case and the crown stop blocking players at that moment, so a player standing on them falls with the rest.

A player killed in the arena skips the countdown of [Death, revive and the spectator](../death/how_death_works.md). They become a spectator `Spectate Delay Seconds` after their death, and nobody can revive them.

A player who joins while the arena is on gets no body and takes no part in the fight. They spectate in free camera until the recap, starting from a player start of the map, not above the platform.

Anything that falls off the platform lands in the void under it. A player there dies. A weapon of the arena there is destroyed.

The HUD keeps health, stamina and the quick slots. `WBP_Gameplay` hides the run clock, the extraction totals and the objective title while the arena runs.

---

## When it starts

Each time a player goes down, the GameMode checks the crew. When every player is down, `BP_FriendslopGameMode` looks for an actor that implements `BPI_Arena` and asks it `CanStartArena`. The first one that answers yes gets `StartArena`, and the run goes on inside it. With no such actor, or when none says yes, the run ends as before.

| Field | What it does | Shipped |
|---|---|---|
| `Min Players for Arena`, on `BP_FriendslopGameMode`, in `Settings|Arena` | Below this many players in the game, a crew that is down ends the run at once. So a solo run never gets the arena | `2` |

`BP_Arena` says yes once per run. After it has run, it stays `Over` until `RESTART` reloads the map.

Every player in the game when the arena starts takes part, spectators included. A player who leaves the game during the arena is taken out of it.

---

## Where it lives

`BP_Arena` is placed in `L_Procedural` itself, far from the map at X `60000`, in the outliner folder `Arena`. Because it is the same map, nothing travels: the GameState, the bank and the recap carry on. The platform, its lights and the void are components of `BP_Arena`. The lights and the void hang from `ArenaCenter`, the root, which never moves, so the void stays under the platform when the core falls. The head, the glass case and the crown are spawned by it when the map starts, on the server.

To add the arena to your own level, drag `BP_Arena` into it, away from the playable space. The players are brought to it.

To remove the arena, delete `BP_Arena` from the level. A crew that is all down then ends the run at once.

---

## The arena fields

All of them are on `BP_Arena`. The timings are in `Settings|Arena`:

| Field | What it does | Shipped |
|---|---|---|
| `Blackout Seconds` | How long the black screen lasts | `3` |
| `Drop Height` | How high above the platform the players appear, in cm | `4000` |
| `Drop Radius` | The radius of the circle they appear on, facing the centre, in cm | `700` |
| `Spawn Protection Seconds` | How long nobody can be hurt, counted from the moment the players appear | `6` |
| `Ring Drop Interval` | Seconds of fighting before the first warning, and between a fall and the next warning | `15` |
| `Ring Warning Seconds` | How long a ring flashes before it falls | `3` |
| `Warning Blink Seconds` | The length of one half of a flash | `0.25` |
| `Ring Fall Seconds` | How long a falling ring stays visible | `2.5` |
| `Ring Fall Gravity` | How fast a falling ring speeds up, in cm/s² | `980` |
| `Win Celebration Seconds` | Seconds between the winner, or `NOBODY WINS`, and the recap | `6` |
| `Spectate Delay Seconds` | Seconds between a death in the arena and the free camera | `1.5` |

What it spawns, also in `Settings|Arena`:

| Field | What it does | Shipped |
|---|---|---|
| `Weapon Loot Classes` | Loot actors that can be picked as a weapon | `BP_Loot_BaseballBat`, `BP_Loot_Hammer` |
| `Weapon Items` | Inventory items that can be picked. They spawn through `Item Pickup Class`, fully charged | `DA_Item_Shotgun_SawedOff` |
| `Extra Weapons` | Weapons on top of one per player. Each is picked at random from both lists | `1` |
| `Weapon Min Radius` | The closest a weapon lies to the centre, in cm. It keeps the weapons off the glass case | `120` |
| `Weapon Radius` | The farthest a weapon lies from the centre, in cm | `430` |
| `Crown Case Class`, `Crown Class`, `Announcer Class` | The glass case, the crown and the head | `BP_CrownCase`, `BP_Crown`, `BP_ArenaAnnouncer` |
| `Crown Height` | The height of the crown above the centre of the platform | `106` |
| `Announcer Offset` | Where the head floats, from the centre of the platform | `(0, 0, 1400)` |

The weapons move every fight, so players cannot learn where they lie. Each one gets a random spot between the two radii and a random turn. They stay spread around the centre. Which weapon spawns is also drawn at random.

The rest:

| Field | What it does | Shipped |
|---|---|---|
| `Intro Message`, `Ring Message`, `Last Standing Message`, `No Winner Message`, in `Settings|Text` | What the head says | `LAST LOSER STANDING`, `THE FLOOR IS LEAVING`, `BREAK THE GLASS`, `NOBODY WINS` |
| `Ring Alarm Sound`, `Kill Sound`, `Win Sound`, in `Settings|Sound` | What the head plays with the ring warning, a kill and the win | `CUE_Arena_RingAlarm`, `CUE_Arena_Laugh`, `CUE_Arena_Fanfare` |
| `Warning Color`, `Warning Off Color`, in `Settings|Style` | The two colours a ring flashes between | red, black |
| `Warning Color Parameter`, in `Settings|Materials` | The colour parameter set on every material of the ring that flashes | `EmissiveColor` |

The kill line `NAME +$100` and the win line `NAME WINS` are `Text` formats in `GetKillMessage` and `GetWinnerMessage`, so they translate like the rest.

The order the floor falls in is set in `BuildDropOrder`: `Ring03`, `Ring02`, `Ring01`, then `PlatformCore`. Take a component out of that list and it never falls.

A piece falls from the centre of the arena. Give a piece you add to the list a relative location of `0`, like the rings. Otherwise it jumps to the centre when it starts to fall. Never add `ArenaCenter`: the void and the lights would fall with it.

---

## The head, the glass and the crown

`BP_ArenaAnnouncer` is the giant head. It shows each message on its screen for `Message Seconds` (`5`), one after the other, and moves its mouth while it talks. On each player's machine it turns toward that player's own camera, at `Turn Speed` (`1.5`). Both are in `Settings|Announcer`.

The head plays its sounds from its face. `CUE_Arena_RingAlarm` uses `ATT_Arena_Announcer`, so the alarm comes from above and is heard at full volume anywhere on the platform. The laugh and the fanfare have no attenuation and sound the same everywhere. Put `ATT_Arena_Announcer` on their Sound Cue to have them come from the head too.

`BP_CrownCase` cannot be broken until one player is left. It implements `BPI_VitalManagerInterface` with `DA_Health_Vital_CrownGlass` (`60` health), so any weapon breaks it, and no weapon needs to know about it. Its shards are `Glass Debris`, in `Settings|Glass`.

`BP_Crown` can be taken with the interact key once the glass is broken, and only by the last player standing. Its prompt is `Interaction Prompt`, `TAKE THE CROWN`. On the winner, it floats at `Worn Offset` (`(0, 0, 125)`, in `Settings|Crown`) above the pawn.

In its case and on the winner, the crown bobs up and down by `Float Height` (`3` cm, in `Settings|Crown`). The Timeline `CrownFloat` plays it. Double-click it to change the curve or its length.

The black screen is `WBP_ArenaIntro`, with `Intro Sound` on it. `Arena Intro Widget Class`, on `BP_FriendslopPlayerController` in `Settings|HUD`, picks it. A replacement has to be a child of `WBP_ArenaIntro`.

---

## Kills and credits

A kill counts for the last player who damaged the victim in the `Kill Credit Seconds` before the death: `5`, on `BP_VitalsSystem`, in `Settings|Vitals`. So a player hit toward the edge who falls into the void still counts for the one who hit them. Dying on your own counts for nobody.

| Field | What it pays | Shipped |
|---|---|---|
| `Kill Reward`, on `BP_FriendslopGameState`, in `Settings|Arena` | Each kill, to the killer | `100` |
| `Win Reward`, same place | The player who takes the crown | `300` |

The money goes to each player's credits at the end of the run, added to their share of the crew's extracted value. It never goes into the team bank, so it does not help the quota.

The recap lists every player under `LAST LOSER STANDING`: their name, their kills, `WINNER` for the one who took the crown, and what the arena paid them. The tag is `Winner Tag`, on `WBP_RunRecap`. See [Change the quota, the run length and the recap](tune_the_run.md).

---

## Who decides

| What | Where it lives | Who decides |
|---|---|---|
| Starting the arena | `BP_FriendslopGameMode` | The server |
| The phase, the fighters, when the floor falls | `BP_Arena` | The server. How much of the floor has fallen is replicated state, so every player sees the same floor |
| The flash of a ring | `BP_Arena` | The server sends the warning, and each machine flashes the ring |
| The glass and the crown | `BP_CrownCase`, `BP_Crown` | The server. Whether the glass is broken and who wears the crown are replicated state |
| The glass case and the crown stop blocking when the core falls | `BP_Arena` (`OnRep_DroppedRings`) | Each machine, when it sees the whole floor gone |
| The bobbing of the crown | `BP_Crown` | Each machine plays it on its own. Nothing is replicated |
| Where the weapons lie | `BP_Arena` | The server. The weapons are replicated actors |
| Kills and the winner | `BP_FriendslopPlayerState` (`Arena Kills`), `BP_FriendslopGameState` | The server |
| The head's messages | `BP_ArenaAnnouncer` | The server sends each message to every player. Each machine shows them in order |
| The black screen | `WBP_ArenaIntro`, on each player's screen | Sent by the server to each player |

---

## Replace it with your own arena

Any actor that implements `BPI_Arena` takes the place of `BP_Arena`. The GameMode only calls three of its functions, on the server:

| Function | When | What yours does |
|---|---|---|
| `CanStartArena` | Every player is down | Return true to take the run. Return false when it has already run or is not ready |
| `StartArena` | Right after, when `CanStartArena` said yes | Start. The GameState already knows the arena has started |
| `RefreshStanding` | A player goes down or leaves while your arena runs. Gives `Ignored Player State`, the player who is leaving | Count who is left and decide what happens |

`NotifyGlassBroken` and `ClaimCrown` are how `BP_CrownCase` and `BP_Crown` talk to `BP_Arena`. Yours can leave them empty.

Your arena then uses the same calls as `BP_Arena`:

1. To show the black screen, call `NotifyArenaStarted` (`BPI_RunControl`) with the seconds it lasts, on every actor that implements `BPI_RunControl`. The player controllers answer it.
2. To bring the players back, copy `ReturnToPlay` from `BP_Arena`. It sets the player back to not down, takes a spectator out of the free camera and restarts the player at the transform you give it.
3. For each kill, call `AddArenaKill` (`BPI_ArenaScore`) on the killer's PlayerState. For the winner, call `SetArenaWinner` (`BPI_RunState`) on the GameState. The credits and the recap follow.
4. When your arena is over, call `NotifyArenaFinished` (`BPI_RunControl`) on the GameMode. It ends the run and opens the recap.

!!! warning
    Your arena has to call `NotifyArenaFinished` in every outcome, a winner, no winner, and every player gone. Until it does, the run never ends and nobody sees the recap. There is no timeout and no error.

---

Next: [How items and the inventory work](../items/how_items_work.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
