# Set up Steam sessions in MPSurvival

After this guide, players host and find games through Steam. The server list shows the Steam name of the host, and players connect through Steam, with no port to open on the router.

MPSurvival hosts, finds and joins games with the engine session nodes in `BP_SurvivalInstance`: **Create Session**, **Find Sessions**, **Join Session** and **Destroy Session**. These nodes talk to Steam as soon as the Steam online subsystem is on, so no graph changes.

## Before you start

- MPSurvival in Unreal Engine 5.5, with the main menu map `MAP_MainMenu`.
- The Steam client, installed, running and logged in.
- To test a join: a second PC with a second Steam account. A Steam account cannot join its own game.

---

## Turn the plugins on

1. Open MPSurvival.
2. Open **Edit**, then **Plugins**.
3. Type `Online Subsystem Steam` in the search box, and tick its box.
4. Type `Steam Sockets` in the search box, and tick its box.
5. Close the editor. Do not open it again yet.

Steam Sockets carries the network driver that connects players through Steam.

---

## Tell Unreal to use Steam

1. Open `Config/DefaultEngine.ini` in a text editor.
2. Add these lines at the end of the file:

    ```ini
    [/Script/Engine.GameEngine]
    !NetDriverDefinitions=ClearArray
    +NetDriverDefinitions=(DefName="GameNetDriver",DriverClassName="/Script/SteamSockets.SteamSocketsNetDriver",DriverClassNameFallback="/Script/OnlineSubsystemUtils.IpNetDriver")
    +NetDriverDefinitions=(DefName="DemoNetDriver",DriverClassName="/Script/Engine.DemoNetDriver",DriverClassNameFallback="/Script/Engine.DemoNetDriver")

    [OnlineSubsystem]
    DefaultPlatformService=Steam

    [OnlineSubsystemSteam]
    bEnabled=true
    SteamDevAppId=480
    bInitServerOnClient=true
    ```

3. Save the file, then open MPSurvival again.

| Line | Why |
|---|---|
| `!NetDriverDefinitions=ClearArray` | Removes the network driver that the engine declares by default. Unreal uses the first driver it finds, so without this line it ignores the Steam driver |
| `SteamSocketsNetDriver` | Players connect through Steam, to a Steam ID |
| `IpNetDriver` | The fallback. Steam does not start in Play In Editor, so the editor keeps playing over IP |
| `DefaultPlatformService=Steam` | Sessions go through Steam instead of the default online subsystem |
| `SteamDevAppId=480` | Spacewar, the test app that Valve shares with every developer. Use your own app ID before you release the game |
| `bInitServerOnClient=true` | Lets a player host a game. Without it, **Host Game** fails |

---

## Search the internet, not the local network

`BP_SurvivalInstance` hosts and searches on the local network by default, because the default online subsystem only works there.

1. Open `Content/MPSurvival/Blueprints/PlayerCharacter/BP_SurvivalInstance`.
2. Click **Class Defaults** in the toolbar.
3. In **Details**, open **Settings**, then **Session**.
4. Untick `Use LAN`.
5. Click **Compile**, then **Save**.

---

## Check that it works

Steam does not start in Play In Editor. Test in a standalone game or in a packaged build.

1. Open `MAP_MainMenu`. A standalone game starts on the map that is open in the editor.
2. Open the menu next to **Play**, and pick **Standalone Game**.
3. Check that the Steam overlay message shows in a corner of the game window.
4. Click **Host Game**, set `Max Players`, then click **Start**.
5. On the second PC, with the second Steam account, start the game the same way.
6. Click **Find Games**. The game of the host shows with the Steam name of the host.
7. Click the row to join.

---

## If it does not work

| What you see | What to check |
|---|---|
| No Steam overlay message | The Steam client runs and is logged in. `Online Subsystem Steam` is ticked in **Plugins** |
| The game starts on `MAP_Showcase`, not on the menu | A standalone game starts on the map that is open in the editor. Open `MAP_MainMenu` first |
| **Find Games** shows `No games found.` | `Use LAN` is unticked in `BP_SurvivalInstance`. With the test app 480, Steam lists only the games of your own region |
| **Host Game** shows `Could not host a game.` | `bInitServerOnClient=true` is in `DefaultEngine.ini` |

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
