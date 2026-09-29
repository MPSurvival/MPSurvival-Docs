# Player colours, nameplates and pings

Every player gets a colour from the server. That colour tints their robot, writes their name above their head for teammates, and paints the pings they place. This page shows where to change the palette, the nameplate and the pings, and how to use the colour on your own actors.

- The colour: `Blueprints/ActorComponents/BP_PlayerColorComponent`, on `BP_FriendslopPlayerState`
- The tint: `Blueprints/ActorComponents/BP_PlayerColorTintComponent`, on `BP_FriendslopCharacter`
- The nameplate: `Blueprints/ActorComponents/BP_NameplateComponent`, on `BP_FriendslopCharacter`
- The ping: `Blueprints/ActorComponents/BP_PingComponent`, on `BP_FriendslopCharacter`, and `Blueprints/Effects/BP_PingMarker`

---

## Change the palette

When a player's PlayerState starts on the server, it takes a random colour from the palette that no other player has. Players do not choose their colour. If there are more players than colours, a colour is used twice.

1. Open `BP_FriendslopPlayerState`.
2. Select `BP_PlayerColorComponent` in the Components panel.
3. Edit the `Colors` array: add, remove or change entries.
4. Save.

| Field | Category | What it does | Default |
|---|---|---|---|
| `Colors` | `Settings|Colors` | The palette | 8 colours: red, orange, yellow, green, cyan, blue, violet, pink |
| `Unassigned Color` | `Settings|Colors` | What the colour reads before the server has assigned one | dark grey |

The body tint is set on the character's `BP_PlayerColorTintComponent`:

| Field | Category | What it does | Default |
|---|---|---|---|
| `Tinted Slot Names` | `Settings|Tint` | The material slots that take the colour, by slot name. Missing slots are skipped | `M_Robot_Shell` |
| `Color Parameter Name` | `Settings|Tint` | The vector parameter written on those slots | `BaseColor` |
| `Tint Strength` | `Settings|Tint` | 0 keeps the material colour, 1 is the full player colour. It is always mixed from the colour the material slot had before any tint, so a rebuild of the body (a cosmetic change) never makes it stronger | `0.55` |
| `Tint From Player Color` | `Settings|Tint` | Off keeps the material colour and only applies patterns | on |
| `Color Source` | `Settings|Tint` | The actor to read the colour from. Empty reads the owner's PlayerState | empty |

---

## Use the colour on your own actor

Everything that shows a player colour asks the PlayerState through `BPI_PlayerColor`, never through a cast. Do the same:

1. Get the player's PlayerState and call `Get Player Color` (`BPI_PlayerColor`) once, when your actor or widget is created.
2. Then bind `OnPlayerColorChanged` on its `BP_PlayerColorComponent` to follow later changes.

Read first, bind second. On a player's machine the colour can arrive before your widget exists, and a widget that only binds never sees it.

On your own PlayerState, add `BP_PlayerColorComponent` and implement `BPI_PlayerColor`. `Get Player Color` returns the component's `Get Color`. `Get Player Color Index` returns its index.

---

## The nameplate

Teammates see your name above your head, in your colour, with a health line. You never see your own. Past `Far Distance`, or behind a wall, the plate fades and shows the distance in metres. It also shows a closed hand while the player carries something, and turns the name red with no health line when the player is down.

The name is the player name of the PlayerState. With the default subsystem, that is the computer name.

Select `BP_NameplateComponent` on `BP_FriendslopCharacter`:

| Field | Category | What it does | Default |
|---|---|---|---|
| `Show Nameplate` | `Settings|Nameplate` | Turns the nameplate on or off | on |
| `Far Distance` | `Settings|Nameplate` | Distance in cm past which the faded version is used | `2500` |
| `Update Interval` | `Settings|Nameplate` | Seconds between two refreshes of distance, occlusion and health | `0.1` |
| `Occlusion Channel` | `Settings|Nameplate` | The trace channel from the camera to the plate. Blocked means faded | `Visibility` |

The component is a Widget Component: its `Widget Class` is `WBP_Nameplate`, drawn in screen space at 240 x 56. Move the component in the Components panel to change the height. The colours are `Health Color`, `Danger Color` and `Distant Color` on `WBP_Nameplate`.

For a voice plugin, call `Set Speaking` on `BP_NameplateComponent` with true while the player talks. The plate then shows a microphone. Nothing in MPFriendslopV2 calls it.

---

## Pings

`Middle Mouse` places a marker where you look (`IA_Ping`). A second press within 0.3 s turns the same marker into a red danger marker (`IA_PingDanger`). The two types are the `Point` and `Danger` entries of `E_PingType`. Markers show through walls with their distance. When they are off screen, they stick to the edge of the screen with an arrow. They follow the object they hit and disappear after 8 seconds. A marker takes the colour of the player who placed it. Danger is red for everyone.

Select `BP_PingComponent` on `BP_FriendslopCharacter`:

| Field | Category | What it does | Default |
|---|---|---|---|
| `Ping Range` | `Settings|Ping` | Length of the trace, in cm | `5000` |
| `Ping Cooldown` | `Settings|Ping` | Minimum seconds between two pings | `0.5` |
| `Server Distance Tolerance` | `Settings|Ping` | The server accepts a ping up to `Ping Range` times this value | `1.25` |
| `Ping Marker Class` | `Settings|Ping` | The marker that is spawned | `BP_PingMarker` |

To restyle the marker, make a child of `BP_PingMarker` and set it in `Ping Marker Class`. `WBP_PingLayer` is the HUD layer that draws the off-screen arrows. It looks for every actor of its own `Ping Marker Class` (`BP_PingMarker`), so it finds your child as it is. Change that field only if your marker does not derive from `BP_PingMarker`. The danger red is `Danger Color` on `WBP_Ping` and on `WBP_PingOffscreen`.

Pings make no sound. Bind `OnPingPlaced` or `OnPingPromoted` to add one.

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnPlayerColorChanged` on `BP_PlayerColorComponent` | The player's colour was assigned. Gives the `Color` | every player |
| `OnPingPlaced` on `BP_PingComponent` | The player pressed ping on something. Gives `Ping Location` | owning player |
| `OnPingPromoted` on `BP_PingComponent` | The player turned their last ping into a danger ping | owning player |

---

## Mistakes that cost time

- **`Discovery Interval` at 0 on `WBP_PingLayer`.** The off-screen arrows stop completely, with no error. Keep it above 0, and check the value on the placed layer, not only on the class.
- **A body part that stays grey.** The tint only reaches material slots named in `Tinted Slot Names`, on the body and on cosmetic meshes alike. Name the slot `M_Robot_Shell`, or add your slot name to the list.

---

Next: [How the menus work, and adding a page](../ui/how_the_menus_work.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
