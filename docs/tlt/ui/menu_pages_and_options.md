# Add or change a menu page

Every menu screen in the template is drawn from a Data Asset. The asset lists the rows of the page, and each row says which widget draws it and what its numbers are. Adding an option, a button, or a whole page is filling in an array. Only one case needs a graph edit, and it is called out below.

- The page class: `Content/TheLastTemplate/Blueprints/DataAssets/Widgets/Menu/BP_SettingsPageDataAsset`
- The pages that ship: `Content/TheLastTemplate/Blueprints/DataAssets/Widgets/Menu/Childs/`
- The row widgets: `Content/TheLastTemplate/Blueprints/Widgets/Menu/Common/`
- The screens that read them: `Content/TheLastTemplate/Blueprints/Widgets/Menu/Screens/`
- The row struct: `Content/TheLastTemplate/Blueprints/Structures/Widgets/S_SettingRow`

---

## The pages that ship

| Page | Screen that reads it | What it holds |
|---|---|---|
| `DA_Page_MainMenu` | `BP_MainMenuWidget` | Continue, New Game, Load Game, Options, Quit |
| `DA_Page_Pause` | `BP_PauseMenuWidget` | Continue, Options, Save Game, Photo Mode, Quit to Main Menu |
| `DA_Page_Options` | `BP_MenuOptionsRootWidget` | the list of the five option pages |
| `DA_Page_Audio` | `BP_MenuSettingsPageWidget` | six volume sliders under one heading |
| `DA_Page_Graphics` | `BP_MenuSettingsPageWidget` | ten quality selectors and a resolution scale slider |
| `DA_Page_Display` | `BP_MenuSettingsPageWidget` | resolution, window mode, VSync, frame limit, HDR, brightness, calibrate |
| `DA_Page_Controls` | `BP_MenuSettingsPageWidget` | the rebindable actions |
| `DA_Page_Language` | `BP_MenuSettingsPageWidget` | one language selector |
| `DA_Page_Brightness` | `BP_MenuBrightnessWidget` | the brightness slider and the HDR toggle of the calibration screen |

A page asset has three fields.

| Field | What it does |
|---|---|
| `Title Key` | The title drawn at the top of the screen. |
| `Rows` | The rows, in the order they appear. |
| `Build From Input` | Ticked on `DA_Page_Controls` only. The page then fills its key bind rows from the player's Enhanced Input mappings instead of taking the key from the array. |

---

## The eight row widgets

`Row Widget Class` is the row type. There is no enum to pick from, you pick a widget class.

| Class | What the player sees |
|---|---|
| `BP_MenuItemWidget` | A plain button. Used by the main menu, the pause menu and the options list. |
| `BP_MenuRowSectionWidget` | A heading. It cannot be selected and is skipped when navigating. |
| `BP_MenuRowSliderWidget` | A slider with a number. |
| `BP_MenuRowSelectorWidget` | Left and right arrows over a fixed list of choices. |
| `BP_MenuRowToggleWidget` | An on and off switch. |
| `BP_MenuRowResolutionWidget` | Arrows over the resolutions the machine reports. The list comes from the engine, not from the row. |
| `BP_MenuRowKeyBindWidget` | A key icon the player can click to rebind. |
| `BP_MenuRowActionWidget` | A line with a chevron that opens another screen. |

---

## The fields of one row

| Field | What it does |
|---|---|
| `ID` | The name the value is stored under, and what a screen matches on to know which row was pressed. |
| `Row Widget Class` | The widget that draws the row. |
| `Label Key` | The text of the row. Every shipped row uses the same string as `ID`. |
| `Desc Key` | The help line shown while the row is selected. `None` on headings. |
| `Min Value`, `Max Value` | The range of a slider. On a selector these are the first and last index, not values. |
| `Step Value` | How far one step moves. `5` on the volume sliders, `1` on every selector. |
| `Default Value` | The value used when the setting has never been set. |
| `Options Keys` | The list a selector walks through, for example `q.low, q.medium, q.high, q.epic`. |
| `Icon` | A key icon. Only the controls page fills it. |
| `Input Action` | Present on the struct. No shipped row sets it. |
| `Requires Restart` | Present on the struct. No shipped row sets it. |

`Label Key` and `Desc Key` are written as dotted keys so a localisation pass can pick them up later. See [Translate the game into another language](translate_the_game.md).

---

## Add an option to a page that already exists

1. Open the page, for example `DA_Page_Audio`.
2. Add an entry to `Rows`.
3. Set `Row Widget Class` to the row type you want.
4. Type an `ID`. Copy it into `Label Key`, and put `<your id>.desc` in `Desc Key`.
5. Fill `Min Value`, `Max Value`, `Step Value` and `Default Value`, or `Options Keys` for a selector.
6. Save and play. The row is there, the player can change it, and the value is written to the settings file.

!!! warning
    A new `ID` is stored and reloaded on its own, but nothing acts on it. `SG_MenuSettings` is what turns an id into an engine call, in `Apply Audio`, `Apply Graphics` and `Apply Display`. Until you add your id to one of those, or read it yourself, the option moves and does nothing.

---

## Make your new option do something

1. Open `Content/TheLastTemplate/Blueprints/Misc/SG_MenuSettings`.
2. Open `Apply Audio`, `Apply Graphics` or `Apply Display`, whichever fits.
3. Add a branch for your id and do the work there.

Anywhere else in your game, call `Get Value` on `SG_MenuSettings` with your id to read the current setting.

---

## Add a whole options page

1. Duplicate `DA_Page_Audio` into the same folder and name it, for example, `DA_Page_Gameplay`.
2. Set `Title Key` and replace the rows with your own.
3. Open `DA_Page_Options` and add a row: `Row Widget Class` is `BP_MenuItemWidget`, `ID` is `opt.gameplay`, `Label Key` the same, `Desc Key` is `opt.gameplay.desc`.
4. Open `BP_MenuOptionsRootWidget` and find the `Switch on String` that turns a row `ID` into a page Data Asset. Add a pin named `opt.gameplay` and plug your new asset into it.
5. Save both assets and play.

Step 4 is the only graph edit in the whole job. Skip it and the entry appears in the list but opens nothing.

---

## Change the main menu and the pause menu buttons

Both are ordinary pages. Reordering, renaming or removing a button is done in `DA_Page_MainMenu` and `DA_Page_Pause` and nowhere else.

A button that does something new needs a target. The screen widget matches the row `ID` and calls the matching function:

- `BP_MainMenuWidget`: `Start New Game`, `Continue Game`, `Open Load Game`, `Open Options`, `Confirm Quit`.
- `BP_PauseMenuWidget`: `Resume Game`, `Save Game Now`, `Open Photo Mode`, `Quit to Main Menu`.

`BP_MainMenuWidget` also has `Refresh Locks`, which locks rows the player cannot use through `Lock Row` on `BP_MenuRowBaseWidget`. That is where Continue and Load Game are disabled when there is no save.

---

## Where a setting is stored, and when it is applied

Settings live in `SG_MenuSettings`, written to the save slot `TLT_MenuSettings`.

| Variable | Holds |
|---|---|
| `Values` | id to number, for every slider, selector and toggle |
| `Text Values` | id to text |
| `Key Binds` | the keys the player rebound |
| `Pending Changes` | what has been changed but not written yet |

`Set Value` writes, `Get Value` reads. On the settings screen, `Apply Now` pushes the page to the engine and `Apply and Save` also writes the file. `Save Now` writes the file on its own. While a page has unapplied changes the prompt bar grows an apply and a reset entry, and leaving the page opens a confirm pop up instead of going back.

The save files for a run are a different subject. See [Save and load a game](saving_a_run.md).

---

## The prompt bar and the two pop ups

`BP_MenuPromptBarWidget` is the strip of key icons at the bottom of every menu screen. Screens fill it in their own `Setup Prompts`, with `Clear Prompts` then one `Add Prompt` per entry.

!!! warning
    A prompt's label is also its click identifier. Two prompts with the same label on the same bar cannot be told apart, and the click goes to the wrong one.

Two pop ups are shared by every screen:

- `BP_MenuModalConfirmWidget`, a yes and no question. Which question it asks comes from `E_MenuConfirmAction`: `QuitGame`, `DeleteSave`, `NewGame`, `ResetSettings`, `DiscardSettings`, `QuitToMainMenu`. Add a value there to add a confirmation of your own.
- `BP_MenuModalInfoWidget`, one message and one OK.

---

## The brightness calibration screen

`BP_MenuBrightnessWidget` is its own screen, not a settings page, but it builds its two rows from `DA_Page_Brightness` like everything else. It shows the three calibration plates `T_UI_Cal_Shadows`, `T_UI_Cal_Midtones` and `T_UI_Cal_Brights` while the player drags the slider.

It is reached from the `display.calibrate` row of `DA_Page_Display`, which is a `BP_MenuRowActionWidget`. If you want an action row that opens a screen of your own, that widget is where the screen to create is chosen.

---

## Open your own level with the loading screen

`Content/TheLastTemplate/Blueprints/Functions/BFL_LoadingLibrary` has the two nodes you need:

- `Open Level with Loading Screen`, which takes a `Level Name`, puts up `BP_LoadingScreenWidget` and opens the level behind it.
- `Show Loading Screen Until Ready`, which puts the screen up without changing level.

Two places name a level today:

- `BP_BootGameMode` has a `Menu Level Name` variable, set to `L_MainMenu`. That is where the game goes on boot.
- `BP_MainMenuWidget` passes a level name to `Launch Slot`. That is the level the main menu starts. Change it to your own map and the whole main menu follows.

---

## Write your own row type

1. Right click `BP_MenuRowBaseWidget` and create a child widget Blueprint.
2. Build the visual in the designer.
3. Override the ones you need: `Setup`, `Refresh Visual`, `Get Label`, `Get Description`, `Is Selectable`, `Accept`, `Navigate Left`, `Navigate Right`, `Commit Value`.
4. Put your class in `Row Widget Class` on a row.

Nothing else has to change. The page builds whatever class the row names.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
