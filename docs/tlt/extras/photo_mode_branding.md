# Put your own logo, frames and filters in photo mode

Everything that ends up burned into a photo is drawn by one post process material, and every list the player picks from lives in a data asset. So your logo, your aspect ratios and your colour looks go in without opening a graph, as long as you stay inside the counts that ship.

Three places hold all of it:

- the picture itself: `Content/TheLastTemplate/Materials/PhotoMode/`
- the lists, the ranges and the defaults: `Content/TheLastTemplate/Blueprints/DataAssets/Widgets/PhotoMode/`
- the logo texture and the two materials: the `BP_PlayerPhotoModeManager` component on `Content/TheLastTemplate/Blueprints/PlayerCharacter/BP_PlayerCharacter`

For what each tab does for the player, see [Photo mode, tab by tab](photo_mode_overview.md).

---

## The two materials

| Material | What it draws | Parameter groups |
|---|---|---|
| `M_PhotoModeGrade` | Brightness, sharpness, saturation, the colour looks, the grain and the colour fringing, the vignette, the aspect ratio bars, the logo stamp | `A - Display`, `B - Filters`, `C - Vignette`, `D - Frames`, `E - Logo`, `F - Screen Effects` |
| `M_PhotoModeLens` | Depth of field and the motion blur | `A - Depth of Field`, `B - Motion Blur` |

The manager makes a dynamic instance of each one when photo mode opens, and pushes the player's values into the parameters above. The two assets are named on the component in the fields `Grade Material` and `Lens Material`, so you can point them at your own copies, but there is no reason to unless you rewrote them.

The bars and the logo are in the material and not in the widget on purpose. The shot re-renders the scene, and a re-render does not capture UMG, so anything drawn in a widget would be missing from the saved file. The composition grid is a widget for the same reason in reverse: it must never end up in the file.

---

## Put your own logo in

The one that ships is `Content/TheLastTemplate/Textures/Widgets/Menu/T_UI_Logo`, white with an alpha channel, 4096 by 2048.

1. Import your logo as a texture. Keep the transparency, the stamp is drawn with the texture's alpha.
2. Open `BP_PlayerCharacter`.
3. Select the `BP_PlayerPhotoModeManager` component in the Components panel.
4. Set `Logo Texture` to your texture.
5. Compile and save.

That is the whole swap. The manager reads the width and the height of the texture and fills `Logo Aspect Value` itself, so a wide logo is placed as a wide logo. `Logo Aspect Value` is in the `No Edit` group, which is runtime state. Leave it alone.

!!! warning "Set it on the component, not in the component Blueprint"
    `BP_PlayerCharacter` carries its own value for `Logo Texture`, and a value set on a placed component wins over the class default. If you open `BP_PlayerPhotoModeManager` and set the texture in its Class Defaults, nothing changes in game and nothing warns you.

The logo has no opacity slider. The player turns it on or off, picks one of four corners and sets its size. It is drawn at full opacity when it is on.

| `ID` | `Label Key` | Row type | Range or options | `Default Value` |
|---|---|---|---|---|
| `photo.logo.enabled` | `Logo` | Toggle | off or on | 0 |
| `photo.logo.corner` | `Corner` | Selector | `Bottom Right`, `Bottom Left`, `Top Right`, `Top Left` | 0 |
| `photo.logo.size` | `Size` | Slider | 0 to 100, step 1 | 50 |

These three rows are the `Rows` array of `DA_Page_Photo_Logo`. The logo keeps a margin of five percent of the screen from the two edges of the corner it sits in. That margin, and the four corners, are read in `ApplyGrade` on the manager: a fifth corner in `Options Keys` would show in the list and place the logo bottom right.

---

## Change the aspect ratio bars

The list is the `Options Keys` of the `photo.frame.index` row in `DA_Page_Photo_Frames`.

| `ID` | `Label Key` | Row type | Range or options | `Default Value` |
|---|---|---|---|---|
| `photo.frame.index` | `Frame` | Selector | `None`, `2.39 Ratio`, `1.85 Ratio`, `4:3`, `1:1` | 0 |
| `photo.frame.colour` | `Colour` | Selector | `Black`, `White` | 0 |
| `photo.frame.intensity` | `Intensity` | Slider | 0 to 10, step 1 | 10 |

Rename an entry and only the label changes, the ratio stays where it is. The four ratios are matched by index in `ApplyGrade`, so a sixth entry draws no bars until you add its ratio there, and reordering the list renames the ratios rather than moving them.

The bars come out on the top and bottom or on the sides, whichever way the player's window differs from the target ratio, so you do not need a separate entry for a tall window.

`Colour` has exactly two entries because the graph reads index 1 as white and everything else as black. A third entry renders black.

---

## Change or add a colour look

Six entries ship in the `photo.filter.index` row of `DA_Page_Photo_Filters`: `None` plus five looks.

| `ID` | `Label Key` | Row type | Range or options | `Default Value` |
|---|---|---|---|---|
| `photo.filter.index` | `Filter` | Selector | `None`, `Noir`, `Sepia`, `Vintage`, `Cold`, `Warm` | 0 |
| `photo.filter.intensity` | `Intensity` | Slider | 0 to 100, step 1 | 100 |
| `photo.filter.hidechars` | `Hide Characters` | Selector | `None`, `Player`, `All` | 0 |

To rename a look, edit `Options Keys`. To change what a look does, open `M_PhotoModeGrade` and work in the group `B - Filters`: the selected entry arrives there as the `FilterIndex` parameter, and the slider as `FilterIntensity`, which sets how much of it is applied.

To add a seventh entry, add it to `Options Keys`, raise `Max Value` to 6, then handle index 6 in the material. Until the material handles it, the entry shows in the list and does nothing.

---

## Retune what a photo starts at

`Default Value` is the value a row opens on. Nothing else has to change when you move it.

| Page asset | `ID` | `Label Key` | Range | `Default Value` |
|---|---|---|---|---|
| `DA_Page_Photo_ScreenEffects` | `photo.fx.chromatic` | `Chromatic Aberration` | 0 to 100 | 0 |
| `DA_Page_Photo_ScreenEffects` | `photo.fx.grain` | `Film Grain` | 0 to 100 | 0 |
| `DA_Page_Photo_ScreenEffects` | `photo.fx.motionblur` | `Motion Blur` | 0 to 100 | 0 |
| `DA_Page_Photo_Vignette` | `photo.vignette.enabled` | `Vignette` | off or on | 0 |
| `DA_Page_Photo_Vignette` | `photo.vignette.size` | `Size` | 0 to 100 | 23 |
| `DA_Page_Photo_Vignette` | `photo.vignette.intensity` | `Intensity` | 0 to 100 | 100 |
| `DA_Page_Photo_Display` | `photo.display.brightness` | `Brightness` | 0 to 100 | 50 |
| `DA_Page_Photo_Display` | `photo.display.sharpness` | `Sharpness` | 0 to 100 | 50 |
| `DA_Page_Photo_Display` | `photo.display.saturation` | `Saturation` | 0 to 100 | 50 |

Brightness, sharpness and saturation open in the middle of their range, since those three go both ways. The effects that only add something open at 0, so a photo is clean until the player asks for it.

---

## Rename a row, add one, remove one

Every page is a `BP_SettingsPageDataAsset` and its `Rows` array holds one entry per line. Each entry has:

| Field | What it does |
|---|---|
| `ID` | The key the manager reads the value under. This is the load bearing field. |
| `Row Widget Class` | The kind of row. `BP_MenuRowSliderWidget`, `BP_MenuRowSelectorWidget` or `BP_MenuRowToggleWidget`, all in `Content/TheLastTemplate/Blueprints/Widgets/Menu/Common/` |
| `Label Key` | The text on the left. The photo pages put plain text here, so type what you want shown. |
| `Desc Key` | The line under the panel title while the row is selected. Plain text as well. |
| `Min Value`, `Max Value`, `Step Value` | The range of a slider, or the index range of a selector, where `Max Value` is the last option |
| `Default Value` | Where the row opens |
| `Options Keys` | The entries of a selector, in order |

Renaming and retuning are free. Adding is not: the manager reads twenty four keys by name, one per shipped row, and a row with an `ID` it does not know is drawn, moves and saves without anything reading it. So a new row is only useful once something reads its key.

Removing a row takes the control away from the player without breaking anything else. If you do not want the logo tab at all, empty its rows, or drop the tab.

---

## Remove a tab, or add one

The tab strip is built from two arrays on `Content/TheLastTemplate/Blueprints/Widgets/PhotoMode/BP_PhotoModeWidget`:

- `Pages`, one page data asset per tab
- `Tab Icons`, one texture per tab, in the same order

The eight shipped icons are in `Content/TheLastTemplate/Textures/Widgets/Icons/PhotoMode/`. Remove a tab by taking its entry out of both arrays. Add one by making a page data asset next to the others, an icon next to the others, and appending both.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
