# How the menus work, and adding a page

Every menu screen in the template, the main menu and the pause menu alike, is a page shown by one router widget, `WBP_MenuRoot`. The router keeps a stack of pages, shows the one on top and hides the others. `Escape` closes the top page and goes back to the one below.

- The menu widgets: `Content/MPFriendslop/Blueprints/Widgets/Menu/`
- The two interfaces: `Content/MPFriendslop/Blueprints/Interfaces/`

A page has no parent class. It is any Widget Blueprint that implements `BPI_MenuPage`. So a screen of your own plugs in without editing the router.

---

## The router

`WBP_MenuRoot` is the only widget added to the viewport. When more than one page is stacked, it draws a back arrow at the top left.

| Field | What it does | Shipped default |
|---|---|---|
| `Home Page Class` | The first page shown | `WBP_MainMenuPage` |
| `Lobby Page Class` | Shown on top of the home page when the player is already in a session, so a player who joined lands in the lobby | `WBP_SessionPage` |
| `Back Key` | Closes the top page | `Escape` |
| `Shade Visible` | Darkens the game behind the pages. The pause menu turns it on | off |

Pages talk to the router through `BPI_MenuNavigation`:

| Function | What it does |
|---|---|
| `OpenPage` | Puts a new page, given by its class, on top of the stack |
| `CloseTopPage` | Closes the top page. When only one page is left, the router fires `OnCloseRequested` instead |

---

## The main menu

`L_MainMenu` holds a `BP_MainMenu` actor. It puts its camera in view and creates `Menu Widget Class` (`WBP_MenuRoot`). Move the actor to frame your shot.

The home page, `WBP_MainMenuPage`, has five entries: `HOST RUN`, `SOLO RUN`, `CUSTOMIZE`, `SETTINGS` and `QUIT`. Each screen it opens is a class you can swap in its Details:

| Field | What it does | Shipped default |
|---|---|---|
| `Game Title` | The big title at the top. Set it to your game's name | `MPFRIENDSLOP V2` |
| `Level To Play` | The map `SOLO RUN` opens | `L_Procedural` |
| `Session Page Class` | The page `HOST RUN` opens | `WBP_SessionPage` |
| `Customization Page Class` | The page `CUSTOMIZE` opens | `WBP_Customization` |
| `Settings Page Class` | The page `SETTINGS` opens | `WBP_SettingsPage` |
| `Extra Entries` | Lines of your own, shown above `QUIT` in this order. Each one is a `Label` and the `Page Class` it opens | empty |
| `Entry Class` | The widget each extra line is made from | `WBP_MenuEntry` |

Every menu line is a `WBP_MenuEntry`, with a `Label`, a `Rest Color`, a `Focus Color`, a `Hover Sound` and a `Click Sound`, and an `OnClicked` dispatcher that gives the line that was clicked. Reuse it for your own buttons and they match the rest.

---

## The pause menu

`P` opens it. It is the same router, created by `BP_FriendslopPlayerController`, with `WBP_PausePage` on top: `RESUME`, `SETTINGS` and `LEAVE THE RUN`.

The game does not stop. The pause screen sits on top of a running game, solo or with friends, and the HUD hides while it is open.

| Field | Where | What it does | Shipped default |
|---|---|---|---|
| `Pause Widget Class` | `BP_FriendslopPlayerController` | The router created for the pause | `WBP_MenuRoot` |
| `Pause Page Class` | `BP_FriendslopPlayerController` | The first page in it | `WBP_PausePage` |
| `Menu Level` | `WBP_PausePage` | The map `LEAVE THE RUN` opens | `L_MainMenu` |
| `Host Left Text` | `WBP_PausePage` | The reason shown to everyone when the host leaves | `THE HOST LEFT THE RUN` |
| `Pause Action` | `WBP_PausePage` | The action whose keys close the pause from inside the page | `IA_PauseMenu` |

`RESUME`, `Escape` and the pause key all close it the same way, through `CloseTopPage`. The page asks Enhanced Input which keys `Pause Action` is on right now, so a pause key rebound in **CONTROLS** closes the pause as well as it opens it.

---

## Add a page

1. Create a Widget Blueprint in `Content/MPFriendslop/Blueprints/Widgets/Menu/`.
2. In the Designer, select the widget's own name at the top of the **Hierarchy** and tick `Is Focusable`.
3. In **Class Settings**, add `BPI_MenuPage` to the implemented interfaces.
4. Implement `OnPageShown`. It gives you `Router`: keep it in a variable.
5. Build your buttons from `WBP_MenuEntry` and bind their `OnClicked`.
6. To open another page, call `OpenPage` on `Router`. To close yours, call `CloseTopPage` on `Router`.

`BPI_MenuPage` has a second function, `OnPageClosing`, called just before the page is removed. Use it for anything to undo when the player backs out.

!!! warning
    A page without `Is Focusable` shows up, but `Escape` does nothing on it and the player is stuck on that screen.

Then decide how the player reaches it:

| You want to | Do this |
|---|---|
| Replace one of the shipped screens | Point its `... Page Class` field at your page. No graph to open |
| Start the menu on your page | Point `Home Page Class` on `WBP_MenuRoot` at it |
| Add a new line to the main menu | Open `WBP_MainMenuPage`, **Class Defaults**, and add an element to `Extra Entries`: its `Label` and your page as `Page Class`. No graph to open |
| Add a line to the pause menu | Edit `WBP_PausePage`: place one more `WBP_MenuEntry` in the Designer and bind its `OnClicked` to `OpenPage` on `Router` |

---

## Where to go next

- [Sessions and the lobby](sessions_and_the_lobby.md)
- [The HUD, and adding to it](the_hud.md)
- [Settings and volume sliders](settings_and_audio.md)
- [Use your own game framework classes](use_your_own_framework.md)

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
