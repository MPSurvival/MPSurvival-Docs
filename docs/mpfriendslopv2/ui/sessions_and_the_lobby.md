# Sessions and the lobby

`HOST RUN` on the main menu opens one page, `WBP_SessionPage`, with two tabs: `JOIN` lists the games you can join, `HOST` opens your own lobby. From the lobby, the host starts the run and everyone travels to `L_Procedural` together.

- The session page and its rows: `Content/MPFriendslop/Blueprints/Widgets/Menu/`
- The session state: `BP_FriendslopGameInstance`, `BP_LobbyGameState` and `BP_FriendslopPlayerState`

The page runs on the default online subsystem, through the engine's own `Create Session`, `Find Sessions`, `Join Session` and `Destroy Session`.

---

## How a lobby works

The lobby **is** `L_MainMenu`, running as a listen server.

1. The host picks the options on the `HOST` tab and clicks `OPEN LOBBY`. The page creates the session, then reloads `L_MainMenu` as a listen server capped at `MAX PLAYERS`.
2. Another player clicks a line on the `JOIN` tab and travels to the host's `L_MainMenu`. The menu sees that they are in a session and opens the lobby for them directly.
3. Each player has a line in the lobby with their colour and a status: `HOST`, `READY`, `PICKING A HAT`, `ENTERING CODE` or `WRONG CODE`. Clicking your own line opens the customization page.
4. The host clicks `START THE RUN`. Everyone travels to `Run Level`.

The back arrow or `Escape` in a lobby leaves the session and reloads the menu.

---

## The host options

Three lines on the `HOST` tab. The host clicks the value to cycle it.

| Option | What it does | When it can change |
|---|---|---|
| `MAX PLAYERS` | How many players the server lets in. Players beyond it are refused as a full server | Before `OPEN LOBBY` only |
| `WHO CAN JOIN` | `ONLINE` or `LAN ONLY`. On the default subsystem both behave the same | Before `OPEN LOBBY` only |
| `PRIVATE CODE` | `OFF`, or a code of letters that every joining player must type | Any time, lobby open or not |

`MAX PLAYERS` cycles from `Min Players` up to `Max Lobby Players`, which is four. The starting value is `Host Options` on `BP_FriendslopGameInstance` (`Max Players` 4, `Lan Only` off), and the host's choice is kept there for the whole session, restarts included.

---

## The private code

With a code set, a player who joins the lobby lands on a gate titled `THIS CREW IS LOCKED`, with a text box, `CONFIRM` and `LEAVE`. The right code puts them in the lobby. A wrong one shows `WRONG CODE` on their line. After too many wrong codes, or too long without typing one, the server sends them back to the menu with a reason.

The code is checked **after** the player connects. The default subsystem cannot attach custom data to a session, so there is no search by code: a locked lobby is listed like any other.

| Field | Where | What it does | Shipped default |
|---|---|---|---|
| `Code Alphabet` | `WBP_SessionPage` | The letters a code is drawn from | `ABCDEFGHJKLMNPQRSTUVWXYZ` (no I, no O) |
| `Code Length` | `WBP_SessionPage` | How many letters | `4` |
| `Max Join Code Attempts` | `BP_FriendslopPlayerState` | Wrong codes allowed before the player is sent back | `3` |
| `Join Code Timeout` | `BP_FriendslopPlayerState` | Seconds allowed to type the code | `60` |

---

## Joining a run already in progress

A started run stays in the `JOIN` list, but the gate only exists in the lobby. When the host has set a code, a player who joins the run gets no text box: the server sends them straight back to the menu, with `Late Join Refused Text` as the reason. Without a code, they join the run.

The gate title and `Late Join Refused Text` both read `THIS CREW IS LOCKED` by default. They are two separate texts, and changing one leaves the other as it is.

| Text | Shown to | Where you change it |
|---|---|---|
| The gate title | A player who joins a lobby that has a code, above the text box | `CodeTitle` in the Designer of `WBP_SessionPage`, its `Text` |
| `Late Join Refused Text` | A player refused from a run in progress, as the reason they are sent back | `BP_FriendslopGameMode` |

---

## Kicking a player

The host clicks another player's lobby line, then `KICK` on `REMOVE FROM THE CREW?`. The kicked player goes back to the menu with `Kick Reason Text` on `WBP_SessionPage` (`THE HOST REMOVED YOU FROM THE CREW`). Nothing stops them from joining again.

---

## The other fields

The words the page swaps while it runs, such as `OPEN LOBBY`, `START THE RUN`, `ONLINE`, `LAN ONLY` and the error lines, are `Text` fields in `Settings|Text`, so you can rename or translate them without opening a graph. The ones below change behaviour.

| Field | Where | What it does | Shipped default |
|---|---|---|---|
| `Run Level` | `WBP_SessionPage` | The map `START THE RUN` travels to | `L_Procedural` |
| `Refresh Interval` | `WBP_SessionPage` | Seconds between two searches on the `JOIN` tab | `5` |
| `Max Results` | `WBP_SessionPage` | How many sessions a search returns | `50` |
| `Min Players` | `WBP_SessionPage` | The lowest `MAX PLAYERS` value | `2` |
| `Max Lobby Players` | `WBP_SessionPage` | How many lobby lines the page makes, and so the highest `MAX PLAYERS` value | `4` |
| `Lobby Row Class` | `WBP_SessionPage` | The widget each lobby line is made from | `WBP_LobbyRow` |
| `Menu Level` | `BP_FriendslopPlayerState` | The map a player is sent back to (kick, wrong code) | `L_MainMenu` |
| `Good Ping Max`, `Fair Ping Max` | `WBP_SessionRow` | Where the ping colour turns from green to amber, then red | `60`, `150` |
| `Unknown Ping` | `WBP_SessionRow` | A ping at or above this value is not shown. Some online subsystems give this number when they cannot measure the ping | `9999` |

A browser line shows the crew, the free slots and the ping. There is no map column: `Create Session` in Blueprint takes no custom settings, so a session cannot announce which map it will play.

---

## Allow more than four players

The lobby makes one `WBP_LobbyRow` per player when the page opens, `Max Lobby Players` of them, and `MAX PLAYERS` never goes above that number. No graph to open.

1. Open `WBP_SessionPage`, **Class Defaults**, and set `Max Lobby Players`, for example `8`.
2. To open lobbies at that size by default, set `Max Players` in `Host Options` on `BP_FriendslopGameInstance` to the same value.
3. Check that your levels have a `PlayerStart` for each player. `L_Procedural` has `4`, in the start room, in the outliner folder `StartModule/PlayerStarts`: add more there.

!!! warning
    A `Max Players` above `Max Lobby Players` lets the extra players in, but the lobby has no line to show them. Keep `Max Players` at or below `Max Lobby Players`.

---

To send the lobby to your own map, see [The maps, and how a game starts](../start/maps_and_game_flow.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
