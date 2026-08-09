# How the HUD, menus and saving fit together

Three things share this chapter and they barely touch each other. The HUD is a set of small widgets, each one owned by the system that needs it. The menus are one family of widgets driven by Data Assets. Saving is three `SaveGame` objects and one game instance that reads and writes them.

This page says which Blueprint owns which screen, what draws on top of what, and what you open when you want to change something. The pages after it do the work.

- The widgets: `Content/TheLastTemplate/Blueprints/Widgets/`
- The menu screens and rows: `Content/TheLastTemplate/Blueprints/Widgets/Menu/`
- The menu pages: `Content/TheLastTemplate/Blueprints/DataAssets/Widgets/Menu/Childs/`
- The save objects and the game instance: `Content/TheLastTemplate/Blueprints/Misc/`

---

## There is no HUD Blueprint

The template has no `AHUD` class and no single widget that holds the play screen. Every element is its own widget, created by whoever needs it. That is why you can delete the ammo counter and the health ring never notices.

| Widget | Created by |
|---|---|
| `BP_LifeCharacterWidget`, `BP_HitCharacterWidget`, `BP_AimWidget`, `BP_RadialWidget`, `BP_DeathScreenWidget`, `BP_SaveIndicatorWidget`, `BP_DevDebugHUD` | `BP_PlayerCharacter`, in `Initialize Widgets` |
| `BP_AmmoCounterWidget` | `BP_PlayerWeaponManager` |
| `BP_ThrowableWidget` | `BP_PlayerThrowableManager` |
| `BP_InventoryWidget` | `BP_PlayerInventoryManager` |
| `BP_InspectWidget`, `BP_InspectReadWidget` | `BP_PlayerInspectManager` |
| `BP_PhotoModeWidget` | `BP_PlayerPhotoModeManager` |
| `BP_WorkbenchWidget` | the `BP_Workbench` you place |
| `BP_InteractWidget` | a Widget Component on every interactable |
| `BP_TipWidget` | the `BP_TipZone` you place |
| `BP_FinishWidget` | a Widget Component on `BP_NPCCharacter` and `BP_ZombieCharacter` |
| `BP_NotificationWidget` | `BFL_NotificationLibrary`, from any graph |
| `BP_LoadingScreenWidget` | `BFL_LoadingLibrary` |
| `BP_PauseMenuWidget` | `BP_PlayerCharacter`, in `Pause Game` |

Five of the widgets the character creates are class fields in its Details panel: `BP Life Character Widget`, `BP Hit Character Widget`, `BP Aim Widget`, `BP Radial Widget` and `BP Death Character Widget`. Point one of them at a child class of your own and the character builds yours instead.

What each element does and which fields are yours is on [Change what the HUD shows](hud_widgets.md).

---

## The menu module

Every menu screen derives from `BP_MenuScreenBaseWidget`, which derives from `BP_MenuBaseWidget`. The base holds what all of them share: the click and hover sounds, access to the saved settings, and the whole save slot API, including `Get Slot Count`, `Launch Slot`, `Delete Slot` and `Find Latest Slot Index`. A screen of your own that derives from it gets all of that for free.

A screen is not a layout. It holds a `BP_SettingsPageDataAsset`, and `Build Rows` creates one row widget per entry in that asset's `Rows` array. Adding an option, a button or a whole page is filling in an array, which is why the same screen class, `BP_MenuSettingsPageWidget`, draws the audio, graphics, display, controls and language pages.

Three screens do their own thing because a list of rows cannot express them: `BP_MenuLoadGameWidget` lists save slots, `BP_MenuBrightnessWidget` is the calibration screen, and `BP_MenuTitleScreenWidget` is the press-any-key screen that `BP_MenuPlayerController` puts up when `L_MainMenu` opens.

How to add a page, a row type or an option is on [Add or change a menu page](menu_pages_and_options.md).

---

## What draws on top of what

Widgets are added to the viewport with a Z order, and the values in the template are spread far apart on purpose.

| Z order | What sits there |
|---|---|
| -1000 | notifications, so they never cover anything |
| 0 | the play HUD, the inventory, the radial menu, photo mode, and every menu screen |
| 5 | the save indicator |
| 100 | the death screen |
| 1000 | the loading screen |
| 50000 | the inspect view |
| 100000 | the pause menu |
| 100001 | the confirm modal opened from the pause menu |

!!! warning "Anything opened from the pause menu needs a Z order above 100000"
    The pause menu is added at 100000 while every other menu screen is added at 0. A modal you open from it at the default Z order draws behind it, and because the pause menu covers the screen it also swallows the clicks. It reads like a broken focus and it is not.

---

## The three save objects

There are three `SaveGame` classes, and they are separate so that reading one never costs you the others.

| Object | Slot on disk | What it holds |
|---|---|---|
| `SG_MenuSettings` | `TLT_MenuSettings` | every option the player set: `Values`, `Text Values`, `Key Binds`. One file for the whole game, shared by every slot. |
| `SG_MenuProfileSlot` | `TLT_Save_<n>` | the header the load screen reads: `Display Name`, `Level Name`, `Play Seconds`, `Save Date`, `Slot Index`, `Used`. |
| `SG_TLTGameSave` | `TLT_Save_<n>_Data` | the run itself: inventory, recipes, collectibles, skills, weapons, ammo, upgrades, throwables, vitals, the player transform and the per level state. |

One slot is two files because of the load screen. It draws a row per slot with a name, a level and a play time, so it loads only the headers. The run itself is never deserialised until the player picks a slot.

`Save Slot Prefix` on `BP_TLTGameInstance` is the start of those names and the only place they are built. Change it and both files follow.

---

## Who saves what

`BP_TLTGameInstance` does the file work and nothing else. It owns `Save Game State`, `Load Game State`, `Save Checkpoint` and `Delete Game State`, and it broadcasts `On Save Started` and `On Game Saved`, which is how the little save icon knows to appear without polling anything.

It does not know what a save contains. It asks the player character once, through `Capture To Save` and `Restore From Save`, and the character asks its own components in an order it decides. So a system you add goes into the character's `Capture To Save`, not into the game instance. That is on [Save and load a game](saving_a_run.md).

Level state is the other half and it works the other way round. `Capture Level State` and `Apply Level State` sweep every actor in the level that implements `BPI_Saveable` and store the result under the level name. Your actor never talks to the save manager, it just implements three functions. That is on [Make your level objects remember what happened](level_state_saveable_actors.md).

---

## I want to change X, I open Y

| I want to | I open |
|---|---|
| Move or restyle something on the play screen | the widget in `Blueprints/Widgets/`, see [Change what the HUD shows](hud_widgets.md) |
| Change which glyph a key prompt draws | `DA_KeyPrompts`, see [Key icons and letting players rebind keys](key_prompts_and_rebinding.md) |
| Add an option, a button or a page to the menus | the `DA_Page_*` asset, see [Add or change a menu page](menu_pages_and_options.md) |
| Change the level New Game opens | `BP_MainMenuWidget`, see [Maps, game modes and how the game starts](../start/maps_and_startup_flow.md) |
| Change how many save slots there are | `Max Save Slot` on `BP_MenuBaseWidget` |
| Change the names the save files get on disk | `Save Slot Prefix` on `BP_TLTGameInstance` |
| Use a save class of my own | `Save Game Class` on `BP_TLTGameInstance` |
| Put a checkpoint in a level | a `BP_SaveZone`, and its `Save Once` field |
| Change how long the save icon stays up | `Hold Seconds`, `Fade Speed` and `Save Sound` on `BP_SaveIndicatorWidget` |
| Make one of my actors survive a reload | `BPI_Saveable` on that actor, see [Make your level objects remember what happened](level_state_saveable_actors.md) |
| Ship the game in another language | see [Translate the game into another language](translate_the_game.md) |

---

## What is keyboard and mouse only today

Two honest limits, so you know what you are starting from.

`IMC_Default` maps no gamepad key, and the prompt icons that ship cover keyboard and mouse. The controls page rebinds keyboard keys. Nothing in the chain is keyboard specific, so a gamepad set is icons and mappings, not new Blueprints.

Menu rows draw their `Label Key` straight to the screen. The keys are written in dotted form, `audio.overall` and the like, so a localisation pass can pick them up, but the project ships no localisation data yet. A page you add reads its own keys on screen until the text is translated.

---

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
