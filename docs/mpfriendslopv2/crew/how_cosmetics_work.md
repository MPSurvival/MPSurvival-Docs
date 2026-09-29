# How cosmetics work

Players earn credits in runs, spend them on hats, accessories and body patterns in the main menu, and wear what they equipped in every match. Each cosmetic is one Data Asset, and one list on the game instance decides which ones exist.

This page is the mental model. The pages after it are the recipes.

---

## Where players buy

`CUSTOMIZE` in the main menu opens `WBP_Customization`, and so does clicking your own line in the lobby. The screen shows the credits, one tab per slot and a grid of tiles. The robot mannequin of the menu level (`BP_CosmeticPreview`) tries on the selected item before you pay. To buy, the player holds the `BUY` button. Leaving the screen puts the saved outfit back on the mannequin.

A player wears one item per slot. The slots are the entries of `E_CosmeticSlot`: `Hat`, `Accessory` and `Pattern`. The tabs are built from that enum, so a new entry is a new tab with no graph to open.

---

## Where credits come from

| Source | How much |
|---|---|
| A fresh profile | `Starting Credits` on `BP_FriendslopGameInstance`, `0` shipped: a new player earns everything in runs |
| The end of a run | The team's extracted value split evenly between the players. Each share is added to that player's credits with `AddCredits` |
| The arena | `Kill Reward` (`100`) per kill and `Win Reward` (`300`) to the winner, on `BP_FriendslopGameState`. Added to that player's share at the end of the run. See [The Last Loser Standing arena](../loot/the_arena.md) |

The run side is on [How a run works](../loot/how_a_run_works.md).

To show an amount on your own customization or recap screen, call `FormatCredits` from `BPFL_Credits`. It takes an integer `Amount` and returns `Credits` as text, with the `$` sign and a space between thousands: `1250` becomes `$ 1 250`. The customization screen and the main menu display credits with it.

---

## The pieces

| Piece | Where it lives | What it does |
|---|---|---|
| `BP_CosmeticDataAsset` | one `DA_Cosmetic_*` per item, in `Blueprints/DataAssets/Cosmetic/Childs/` | Name, slot, icon, `Price`, mesh and socket, or pattern |
| `Cosmetic Catalog` | `BP_FriendslopGameInstance`, `Settings\|Cosmetics` | The only list of cosmetics. The shop shows it, the server checks against it |
| `BP_CosmeticComponent` | `BP_FriendslopPlayerState` | Holds what the player has equipped |
| `BP_CosmeticVisualsComponent` | `BP_FriendslopCharacter`, and the menu mannequin | Attaches the equipped meshes to their sockets |
| `BP_PlayerColorTintComponent` | `BP_FriendslopCharacter`, and the menu mannequin | Draws the player colour and the equipped pattern on the body. The mannequin keeps its own colour |
| `SG_FriendslopProfile` | one save slot on each player's machine | Credits, owned cosmetics and the saved outfit |

Hats and accessories attach to a socket of the character mesh: `HatSocket` on the `head` bone, `BackSocket` on `spine_04`. The socket is the offset. There is no offset field. The meshes copy the visibility flags of the body they sit on, so you never see your own hat in first person.

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnCosmeticsChanged` on `BP_CosmeticComponent` | The equipped set changed, or arrived | every player |

---

## The profile and your own save

Credits and ownership go through `BPI_PlayerProfile`, which `BP_FriendslopGameInstance` implements. `GetCredits`, `TryPurchase`, `IsCosmeticOwned`, `AddCredits` and the rest all read and write `SG_FriendslopProfile`, in the slot named by `Profile Slot`. The profile is loaded when the game starts and written again on every purchase, outfit change, settings change and credit gain.

To keep the profile somewhere else, reimplement `SaveProfile` and `LoadProfile` on your game instance. Nothing else in the template needs to change.

---

## On your own character

1. Add `BP_CosmeticComponent` to your player state.
2. Add `BP_CosmeticVisualsComponent` and `BP_PlayerColorTintComponent` to your pawn. `Target Mesh` left empty dresses the first skeletal mesh of the actor.
3. Add `HatSocket` and `BackSocket` to your skeletal mesh, or use your own socket names in the Data Assets.
4. If you use your own game instance, implement `BPI_PlayerProfile` on it and give it the catalogue. See [Use your own game framework classes](../ui/use_your_own_framework.md).

To add a hat, an accessory, a pattern or a new slot, see [Add a hat, an accessory or a pattern](add_a_cosmetic.md). Emotes and faces are on [Add an emote or a face](add_an_emote_or_a_face.md), player colours on [Player colours, nameplates and pings](colours_nameplates_and_pings.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
