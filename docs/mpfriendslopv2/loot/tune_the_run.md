# Change the quota, the run length and the recap

The quota and the length of a run are two numbers on the GameMode. The clock and the end-of-run screen are two widgets you restyle like any other. None of it needs a graph. If you have not read [How a run works](how_a_run_works.md), start there.

- The GameMode: `Content/MPFriendslop/Blueprints/PlayerCharacter/BP_FriendslopGameMode`
- The clock and the recap: `Content/MPFriendslop/Blueprints/Widgets/Gameplay/`

---

## Change the quota and the run length

1. Open `BP_FriendslopGameMode`.
2. Click **Class Defaults**.
3. In `Settings|Extraction`, set `Run Quota` and `Run Duration Seconds`.
4. Compile and save.

| Field | What it does | Shipped |
|---|---|---|
| `Run Quota` | The extracted value the crew must sell before the trap opens for good and everyone can jump | `2000` |
| `Run Duration Seconds` | How long the run lasts. When it runs out, the run ends | `300` (5 minutes) |
| `Starting Extracted Value` | Money already in the bank when the run starts. It shows on the recap as a `STARTING FUNDS` row | `0` |

`Run Quota` is the quota of a map without a level generator. On `L_Procedural` the quota follows the loot that spawned instead, through `Quota Ratio` on the same GameMode. See [Tune the generator](../world/tune_the_generator.md).

`Starting Extracted Value` is a test shortcut. Set it at or above `Run Quota` and the quota is met the moment you spawn, so you can test the trap and the jump without carrying anything. Put it back to `0` for your game.

---

## Different numbers on each map

`L_Procedural` and `L_Showcase` have no GameMode override: they use the project default, `BP_FriendslopGameMode`, set in **Project Settings**, then **Maps & Modes**. So changing its Class Defaults changes every map.

For a map with its own quota or length:

1. Right click `BP_FriendslopGameMode`, then **Create Child Blueprint Class**.
2. In the child's Class Defaults, set `Run Quota` and `Run Duration Seconds`.
3. Open your map, then **World Settings**, and set `GameMode Override` to the child.

---

## The clock

`WBP_RunClock` shows the time left as `M:SS`. It turns to its warning colour when little time is left.

| Field | What it does | Shipped |
|---|---|---|
| `Warning Seconds` | Below this many seconds left, the clock switches to `Warning Color`. In `Settings|Clock` | `60` |
| `Normal Color` | The clock's colour for the rest of the run. In `Style` | amber |
| `Warning Color` | The clock's colour under `Warning Seconds`. In `Style` | red |

Open `WBP_RunClock`, click **Class Defaults**, and change them there. Fonts and sizes are in its Designer. Where the clock sits on screen is covered in [The HUD, and adding to it](../ui/the_hud.md).

---

## Restyle the recap

The recap is `WBP_RunRecap`. It opens by itself on every player's screen when the run ends. Each line of the manifest, of the crew and of the arena is a `WBP_RecapRow`: change its layout and fonts in its Designer.

After a [Last Loser Standing arena](the_arena.md), a `LAST LOSER STANDING` block sits under `CREW`: one row per player with their kills, the `WINNER` tag and what the arena paid them. Without an arena the block is hidden.

When there are more rows than fit, the lists scroll: the manifest on the left (`ManifestScroll`), and `CREW` with `LAST LOSER STANDING` on the right (`PayoutScroll`), under the total and the bar, which stay in place. The recap always opens scrolled to the top.

Every text and colour of the recap is in the Class Defaults of `WBP_RunRecap`, in `Settings|Text` and `Settings|Style`:

| Field | What it does | Shipped |
|---|---|---|
| `Success Headline` | The title when the run met the quota and at least one player jumped out | `YOU MADE IT OUT` |
| `Failure Headline` | The title in every other case, a crew that met the quota and then died included | `QUOTA NOT MET` |
| `Success Color` / `Failure Color` | The colour of each title | |
| `Label Color` / `Value Color` | The colours of the row names and of the amounts | |
| `Left Behind Label` | The line for the value still in the map when the run ended | `LEFT BEHIND` |
| `Lost Color` | The colour of that line | |
| `Restart Label` | The crew leader's button: the host, or on a dedicated server the player connected the longest | `RESTART` |
| `Waiting for Server Label` | What the other players read instead of the button | `WAITING FOR THE SERVER TO RESTART` |
| `Disabled Opacity` | How faded the `RESTART` button gets once clicked | `0.35` |
| `Winner Tag` | Added after the arena winner's kills | `  -  WINNER` |

Every label is a `Text`, so it can be translated.

---

## Replace the recap with your own screen

The screen is picked by `Recap Widget Class`, on `BP_FriendslopPlayerController`, in `Settings|HUD`. It ships set to `WBP_RunRecap`. Set it to your own widget and the controller adds yours to each player's screen instead.

Your screen then has to do what `WBP_RunRecap` does:

1. Start hidden, and show itself when the run ends. `GetRunState`, on the GameState's `BPI_RunState` interface, tells you whether the run has ended, and the GameState's dispatcher `OnRunPhaseChanged` fires when that changes.
2. Read the run through `BPI_RunState`: `GetRunRecap` gives the extracted value, the quota, the left-behind value, the manifest and whether at least one player made it out, and `GetCrewShare` gives each player's share of the extracted value. For the arena, `GetArenaState` gives whether it took place and its winner, `GetArenaReward` what it paid one player, and `GetArenaKills` (`BPI_ArenaScore`, on each PlayerState) their kills.
3. Send the restart through `BPI_RunControl`, `RequestRestart`, on the player's own PlayerState. Show the button to the crew leader only: `GetCrewLeader`, on `BPI_RunState`, returns their PlayerState. The server refuses the request from anyone else.

!!! warning
    The controller hides the HUD and stops gameplay input only for a `WBP_RunRecap`: it listens to that widget's dispatcher `OnRecapOpenChanged`. With a widget of your own, the HUD stays on screen and gameplay input is not stopped while your screen is open. There is no error.

---

Next: [The Last Loser Standing arena](the_arena.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
