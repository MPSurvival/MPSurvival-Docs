# Key icons and letting players rebind keys

No prompt in this template picks its own texture. A widget hands over an **Input Action**, the template looks up which key is bound to that action right now, and draws the matching picture. Change the key in `IMC_Default`, or let the player change it in the options menu, and every prompt on screen follows without you touching a widget.

Three assets do the whole job:

- `Content/TheLastTemplate/Blueprints/Functions/BFL_InputLibrary`, which turns an Input Action into a key and a picture.
- `Content/TheLastTemplate/Blueprints/DataAssets/Widgets/KeyPrompts/Childs/DA_KeyPrompts`, the list that says which picture belongs to which key.
- `Content/TheLastTemplate/Textures/Widgets/KeyPrompts/x80/`, the pictures themselves.

If you have not read [Change the controls](../start/change_the_controls.md) yet, read it first. It covers `IMC_Default` and the actions that ship.

---

## Inside `DA_KeyPrompts`

`DA_KeyPrompts` is a `BP_KeyPromptSetDataAsset`. It holds three arrays that are read at the same index.

| Field | What it holds |
|---|---|
| `Key List` | The key itself, as Unreal names it: `E`, `LeftShift`, `SpaceBar`, `RightMouseButton`, `MouseWheelAxis` |
| `Icon List` | The picture drawn for that key |
| `Name List` | The short text label for that key |

The set covers keyboard and mouse. Every picture in `x80` is 80 pixels tall and between 80 and 220 pixels wide, because a key like `SPACE` needs more room than `E`. That is why a prompt has to keep the ratio when it builds its brush, instead of dropping the texture into a square box.

A number of pictures ship without a row in `DA_KeyPrompts`, among them `T_Key_0` to `T_Key_9`, `T_Key_At`, `T_Key_Question`, `T_Key_Euro` and `T_Mouse_Any`. They are there for you to use.

---

## Add a picture for a key that has none

1. Put your texture in `Content/TheLastTemplate/Textures/Widgets/KeyPrompts/x80/`. Keep the height at 80 and let the width grow with the label.
2. Open `DA_KeyPrompts`.
3. Add one entry to `Key List` and pick the key.
4. Add one entry to `Icon List` and one to `Name List`, at the same index.
5. Save.

The three arrays are read by index, so they must always have the same length and the same order. Add to one and forget the others and every key after that point draws the wrong picture, with no warning.

---

## Ask for a prompt from your own Blueprint

`BFL_InputLibrary` has two functions. Both are Blueprint function library calls, so they work from any widget, actor or component.

| Function | In | Out |
|---|---|---|
| `Get Action Input Informations` | `Action`, `World Context`, `Prompt Set` | `Key`, `Icon`, `Display Name`, `Found` |
| `Get Key Prompt Infos` | `In Key`, `Prompt Set` | `Icon`, `Display Name`, `Found` |

Use the first one in almost every case: it reads the binding that is live for the player. Use the second only when you already hold a key and just want its picture.

To show a prompt in a widget of your own:

1. Add a variable of type `BP_KeyPromptSetDataAsset`, name it `Prompt Set`, and set its default to `DA_KeyPrompts` in the Details panel.
2. Add a variable of type `Input Action` for the action you want to draw.
3. Call `Get Action Input Informations`, feeding it that action, `self` as the world context, and your `Prompt Set`.
4. Branch on `Found`. When it is false the action has no key at all, so hide the image and leave the text alone.
5. Build the brush from `Icon` with a width of `SizeX * Height / SizeY`, so the picture keeps its shape.

Neither the action nor the prompt set is written into the graph as a path. They are variables you fill in the Details panel, which also means a widget you build yourself shows nothing until you fill them.

Three widgets already do this and are worth opening as examples: `BP_TipWidget` and its `Set Tip` function, `BP_FinishWidget` with its `Finisher Action` and its `Refresh Key Icon`, and `BP_ShowcasePanelWidget`.

`BP_FinishWidget` resolves its picture when the widget is built. If the player rebinds mid game the prompt keeps the old picture until something calls `Refresh Key Icon` again, which is why that function is public.

!!! tip
    You do not need a Blueprint at all for a simple on screen hint. Drop a `BP_TipZone` in the level, set `Tip Action` and `Tip Message`, and the message appears with the right key picture while the player stands inside the box.

---

## The Controls page in the options menu

The options menu already has a Controls page, and it really rebinds: the new key goes through Enhanced Input's player mappings and is written to `SG_MenuSettings` under `Key Binds`, so it survives a restart.

The page is a Data Asset, `Content/TheLastTemplate/Blueprints/DataAssets/Widgets/Menu/Childs/DA_Page_Controls`. Its `Rows` array holds two heading rows and seventeen rebindable ones. Each rebindable row uses these fields of `S_SettingRow`:

| Field | What it does |
|---|---|
| `ID` | The name of the mapping in `IMC_Default`. This is the only link between the row and the key |
| `Row Widget Class` | `BP_MenuRowKeyBindWidget` for a rebindable line, `BP_MenuRowSectionWidget` for a heading |
| `Label Key` | Text key for the action name, for example `ctrl.act.moveforward` |
| `Desc Key` | Text key for the help line. Every shipped row uses `ctrl.act.desc` |
| `Options Keys` | The default key, written as its Unreal key name: `Z`, `LeftShift`, `SpaceBar`, `RightMouseButton` |
| `Icon` | The picture that goes with that default key |

`Label Key` and `Desc Key` are text keys, not text. See [Translate the game into another language](translate_the_game.md).

!!! warning
    `ID` must match, letter for letter, the `Name` inside the `Player Mappable Key Settings` of the matching row in `IMC_Default`. If it does not, the line still appears in the menu and the player can still press a key, and nothing at all gets rebound.

Two shipped rows are in exactly that state: `ChangeLocomotion` and `ShoulderSwap` have no mapping of that name in `IMC_Default`. They draw, they do not rebind. Give them a mapping name or delete the rows.

---

## Add your own action to the Controls page

Say you added `IA_Whistle` and want the player to be able to move it.

1. Open `Content/TheLastTemplate/Inputs/IMC_Default` and find the mapping row for `IA_Whistle`.
2. Set its `Setting Behavior` so the row carries its own `Player Mappable Key Settings`, then fill in `Name`, `Display Name` and `Display Category`.
3. Open `DA_Page_Controls` and add a row to `Rows`.
4. Set `Row Widget Class` to `BP_MenuRowKeyBindWidget`.
5. Set `ID` to the same `Name` you typed in step 2.
6. Set `Label Key` and `Desc Key` to your text keys.
7. Set `Options Keys` to the default key name and `Icon` to its picture from `x80`.
8. Save both assets.

Nothing else. You do not open a widget, and you do not touch the menu screens. Adding or reordering pages is covered in [Add or change a menu page](menu_pages_and_options.md).

---

## What the player sees while the game waits for a key

Activating a row opens `BP_MenuRebindWidget` over the page. It takes the keyboard focus and the next key pressed becomes the new binding. Its prompt bar shows `Escape` to back out without changing anything.

The small key chips in that bar, and in every menu footer, are `BP_MenuKeyPromptWidget` inside a `BP_MenuPromptBarWidget`. They are the same pictures from `x80`, so a menu prompt and an in game prompt always look alike.

---

## Keyboard and mouse only

`DA_KeyPrompts` has no gamepad entries, and there are no gamepad pictures in `x80`. Because of that, `Get Action Input Informations` deliberately skips a pad: when an action has more than one key bound and the first one is a gamepad key, it takes the second. A pad binding therefore never blanks out the prompt for a keyboard player.

If you want pad prompts:

1. Draw the pad pictures and import them the same way, one height, variable width.
2. Make a second `BP_KeyPromptSetDataAsset` next to `DA_KeyPrompts` and fill its three arrays with pad keys.
3. Point each widget's `Prompt Set` at the set that matches the device in use.
4. Change the key choice inside `Get Action Input Informations`, which today always prefers the key that is not on a pad.

The Controls page is keyboard and mouse too. It rebinds the mapping name, so a pad binding on the same action is not shown and not touched.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
