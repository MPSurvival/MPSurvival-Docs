# How multiplayer works

MPFriendslopV2 runs as a **listen server**. One player hosts: their game is the server, and the other players join it as clients. The host plays like everyone else.

A run started with `SOLO RUN` opens the map without the listen option, so nobody can join it. Hosting goes through `HOST RUN`, which is covered in [Sessions and the lobby](../ui/sessions_and_the_lobby.md).

---

## What the server decides

The server holds the truth for everything that changes the game. A client asks, the server checks, and the result reaches every player.

| What | Decided by | What every player sees |
|---|---|---|
| Movement | The server, through Character Movement. The moving player predicts locally | Every player's position |
| Health and damage | The server applies damage and writes the vitals | The current health and stamina values |
| Doors, buttons and levers | The server, after checking that the player is in reach | The door in its current state |
| Grabbing | The server accepts or refuses the grab, and decides when it breaks | Who holds what, and where it is |
| Loot value, breaks and sales | The server measures every hit | The item's condition, its break, its sale |
| Quota and extracted value | The server, on the GameState | The same totals for everyone |
| Enemies | The enemy's brain runs on the server only | The enemy's body and state |
| Worn cosmetics | The server | Each player's outfit |

---

## What each player draws for themselves

Some things never cross the network. Each machine builds them on its own, for its own player:

- the camera feel (bob, landing, crouch),
- the HUD, the interaction prompt and the highlight on what you look at,
- the walking and running footsteps, and the damage feedback,
- the key icons, and the key rebinding,
- the run clock, counted by each machine from one shared end time,
- the pieces of a broken item, which land in different places on each machine.

Credits and the cosmetics a player owns are saved on that player's own machine. What they wear is shared.

---

## Players who join late

A player who joins in the middle of a run receives the **state** of the game, not its history. An open door, a damaged item, a dead player's head or an outfit reach them, because each one is a replicated value. A one-off event, like the flash and sound of an old impact, is not replayed. That is intended.

A player who joins late sees the loading screen until their own modules are visible. The run is not restarted for them.

A player who connects while the end-of-run recap is on screen does not spawn: they spectate until the next run. The same goes for a player who connects during [the arena](../loot/the_arena.md): they spectate until the recap.

The lobby holds four players by default. The cap is `Max Players`, chosen by the host before the lobby opens.

---

## Rules for your own additions

1. **Change a state on the server.** A value changed on a client stays on that client.
2. **A state that lasts is a `Replicated` variable, not an event.** A `Multicast` sent before a player joined never reaches that player.
3. **A `Run on Server` event checks what it receives.** The template's own requests check distance and permission before acting. Do the same, or any client can call yours with anything.

On a listen server, the host is both the server and a locally controlled player. So `Has Authority` and `Is Locally Controlled` are both true there, and a mistake between the two never shows on the host.

For logic meant for one player only, check `Is Locally Controlled` at the moment the logic runs, not once in `BeginPlay`. A respawned pawn gets its controller after its `BeginPlay`.

Dispatcher tables in this manual have a **Fires on** column: `server`, `every player`, or `owning player` (the player who acted, on their own machine). Bind to the one that matches where your reaction must run.

---

## Testing with two players

Always judge from the **client** window, never from the host's. The host is the server, so an authority mistake works there and breaks only on a client.

**The quick test, in the run map:**

1. Open `L_Procedural` (it is the editor startup map).
2. In the **Play** menu, set **Number of Players** to `2` and **Net Mode** to `Play As Listen Server`.
3. Press Play and look at the client window.

This skips the menu and the lobby. It is enough for everything that happens during a run.

**The full test, from the menu:**

1. In the **Play** menu, open **Advanced Settings** and untick `Run Under One Process`.
2. Open `L_MainMenu` and press Play with two players.
3. Host with `HOST RUN` in one window, join from the other.

Use this one for the private code, the shop terminal and anything saved. Two players in one process share the session search, the mouse focus on world screens and the save slot. In that setup clicks on the shop screen do nothing and credits look wrong.

On a single PC, with the default online subsystem, the browser of one window does not list the session hosted by another window, even in two processes. Join it with the console command `open 127.0.0.1` instead, and test the browser itself on two PCs.

Starting `L_MainMenu` as a listen server does not put you in a lobby: you land on the main menu and host from there.

**The dedicated server test:**

1. Open `L_Procedural`.
2. In the **Play** menu, set **Net Mode** to `Play As Client` and **Number of Players** to `2` or more.
3. Press Play. The editor runs a dedicated server with no window, and every window is a client.

There is no host on a dedicated server. The `RESTART` button belongs to the crew leader, the player connected the longest, and passes to the next one when that player leaves. The server has no screen, so `OnModulesVisible` never fires on it.

In the editor, the dedicated server plays the map that is open. A server started without a map loads **Server Default Map** (Project Settings > Maps & Modes), which is `L_Procedural`. Packaging a dedicated server needs a source build of Unreal Engine and a C++ project with a Server target: see [Setting Up Dedicated Servers](https://dev.epicgames.com/documentation/en-us/unreal-engine/setting-up-dedicated-servers-in-unreal-engine) in the Unreal documentation.

`START THE RUN` and `RESTART` change map with seamless travel, which the editor turns off in PIE by default. `Config/DefaultEngine.ini` turns it back on with `net.AllowPIESeamlessTravel=1`, under `[ConsoleVariables]`, so the other players follow the host in PIE as they do in a packaged game. Keep that line if you move these maps into a project of your own.

---

## Values on placed actors

!!! warning
    A value typed on a placed actor wins over the class default. And a variable added to a component after that component was placed keeps the empty value of its type (`0`, `false`, `None`) on every copy already placed, with no error. After you change or add a field, open the Details of the placed actors and of each Blueprint that carries the component, and check the value.

---

Next, see [The maps, and how a game starts](maps_and_game_flow.md) and [The controls, and adding an input](controls.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
