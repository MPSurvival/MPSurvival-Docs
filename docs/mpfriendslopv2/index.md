# MPFriendslopV2

Welcome to the documentation for **MPFriendslopV2**, a first person co-op extraction horror template for Unreal Engine.

You and your crew drop into a dark map to loot it. Loot spawns on floors and shelves, and you grab it and carry it with physics, so every hard hit lowers its price and a fragile piece can shatter. You sell it at the extraction trap, spend what the team has sold on supplies from the shop terminal, and keep clear of the Warden, a robot that patrols, hears the noise you make and fires at you. Once the team has sold enough to meet the quota, everyone jumps into the shaft to get out, and the recap shows what was sold, what was left behind and each player's share.

A player who dies loses their head, and the others have until the end of a countdown to carry it to a revive bay. When the whole crew is down, everyone falls into an arena in the dark and fights until one player is left to take the crown. Credits earned in runs buy hats, accessories and body patterns in the main menu, and every player wears their outfit in the match. The player character has no variables of its own: everything it does is a component that can go on your own pawn too. Almost everything you will want to change is a **Data Asset** or a field in the **Details panel**, not a graph.

---

## How do I...

| I want to | Page |
|---|---|
| Add a loot item that sells | [Add a loot item](loot/add_a_loot_item.md) |
| Make my own actor grabbable | [Make any actor grabbable](grab/make_an_actor_grabbable.md) |
| Make any actor usable with the interact key | [Make any actor interactable](interaction/make_an_actor_interactable.md) |
| Open a door with a button or a lever | [Link a button or a lever to a door](interaction/doors_buttons_and_levers.md) |
| Make a button switch my own actor | [Make your own door, switch or lever](interaction/make_your_own_door_or_lever.md) |
| Add an item to the inventory | [Add an inventory item](items/add_an_item.md) |
| Add an item to the shop | [Add an item to the shop](items/the_shop.md) |
| Add a new gun | [Add a new gun](weapons/add_a_new_gun.md) |
| Turn a prop into a melee weapon | [Turn any grabbable object into a melee weapon](weapons/make_a_melee_weapon.md) |
| Make a new enemy | [Make an enemy variant or your own enemy](ai/make_your_own_enemy.md) |
| Change the quota and the length of a run | [Change the quota, the run length and the recap](loot/tune_the_run.md) |
| Change, remove or replace the arena when the whole crew is down | [The Last Loser Standing arena](loot/the_arena.md) |
| Add a hat or a pattern | [Add a hat, an accessory or a pattern](crew/add_a_cosmetic.md) |
| Add an emote | [Add an emote or a face](crew/add_an_emote_or_a_face.md) |
| Add a hand item like the flashlight | [Make a held item, its hold pose and hand IK](items/make_a_held_item.md) |
| Use my own character | [How the player character works](player/how_the_player_works.md#put-it-on-your-own-character) |
| Build my own map | [Build your own level](world/build_your_own_level.md) |
| Play my own map from the menu | [The maps, and how a game starts](start/maps_and_game_flow.md) |
| Test with two players | [How multiplayer works](start/how_multiplayer_works.md) |

---

## Where to start

1. [The maps, and how a game starts](start/maps_and_game_flow.md)
2. [How multiplayer works](start/how_multiplayer_works.md)
3. [The controls, and adding an input](start/controls.md)

---

## The chapters

| Chapter | What it covers |
|---|---|
| [Getting started](start/maps_and_game_flow.md) | The maps, the flow from the main menu to the recap, the listen server model and the keys |
| [The player character](player/how_the_player_works.md) | The components of the pawn, movement, sprint and stamina, the camera, health and damage, and your own character |
| [Interaction, doors and levers](interaction/how_interaction_works.md) | The interact key, making an actor usable, buttons and levers that drive doors, and your own switches |
| [Grabbing and carrying](grab/how_grabbing_works.md) | Physics grabbing, making an actor grabbable, and the weight |
| [Loot and extraction](loot/how_a_run_works.md) | The run loop: loot spawns, value and damage, breakable items, the extraction trap, the quota, the recap and the Last Loser Standing arena |
| [Items, inventory and the shop](items/how_items_work.md) | Item Data Assets, the quick slots, held items with their hold pose, and the shop terminal |
| [Weapons and melee](weapons/how_weapons_work.md) | The gun, adding a new gun, and swinging a grabbed object as a melee weapon |
| [Enemies](ai/how_the_enemy_ai_works.md) | The Warden, its patrol, what it sees and hears, and your own enemy |
| [Death, revive and spectating](death/how_death_works.md) | The lost head, the countdown, the revive bay and the free camera spectator |
| [Cosmetics, emotes and the crew](crew/how_cosmetics_work.md) | Hats, accessories, patterns and credits, emotes and faces, player colours, nameplates and pings |
| [Menus, lobby, HUD and settings](ui/how_the_menus_work.md) | The menu router, sessions and the lobby, the HUD, the settings screen and your own framework classes |
| [Building a level](world/how_levels_are_built.md) | The modular kit and its grid, building a run map, lamps, sounds and footsteps |

---

## How it is played

- **Co-op**, on a **listen server**: one player hosts and plays, the others join. The lobby holds four players by default, set by `Max Players` in `Host Options` on `BP_FriendslopGameInstance`.
- **Every displayed string is a `Text`**, so the widgets are yours to translate.
- **The keys live in `IMC_Default`.** Dragging a mapping onto another key takes about five seconds.
- **Keyboard and mouse.** No context ships a gamepad mapping.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
