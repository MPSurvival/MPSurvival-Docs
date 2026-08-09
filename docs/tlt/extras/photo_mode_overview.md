# Photo mode, tab by tab

Photo mode stops the world, hands the player a free camera, and lets them tune the picture over eight tabs before saving a high resolution shot.

It is one component on the player, one camera actor, one widget, and eight Data Assets that describe the tabs.

- The component: `Content/TheLastTemplate/Blueprints/ActorComponents/BP_PlayerPhotoModeManager`, already on `BP_PlayerCharacter`
- The camera: `Content/TheLastTemplate/Blueprints/PhotoMode/BP_PhotoModeCamera`
- The panel: `Content/TheLastTemplate/Blueprints/Widgets/PhotoMode/BP_PhotoModeWidget`
- The tabs: `Content/TheLastTemplate/Blueprints/DataAssets/Widgets/PhotoMode/`

---

## Two ways in

There are two entry points, and both call `Toggle Photo Mode` on the component:

- the **Photo Mode** row of the pause menu, which is the `pause.photomode` row of `Content/TheLastTemplate/Blueprints/DataAssets/Widgets/Menu/Childs/DA_Page_Pause`
- the **Photo** entry of the [developer menu](developer_menu.md), which closes the menu and opens photo mode

No key opens photo mode on its own. If you want one, make an Input Action, map it in `IMC_Default`, and call `Toggle Photo Mode` on `BP_PlayerPhotoModeManager` from the player character. See [Change the controls](../start/change_the_controls.md).

---

## The eight tabs

The tab order is fixed by the `E_PhotoModeTab` enumeration, and each tab reads one Data Asset.

| Tab | Data Asset | What it controls |
|---|---|---|
| Camera | `DA_Page_Photo_Camera` | Free camera or gameplay camera, tilt, field of view |
| Depth of Field | `DA_Page_Photo_DepthOfField` | The blur, and what stays sharp |
| Display | `DA_Page_Photo_Display` | Brightness, sharpness, saturation |
| Screen Effects | `DA_Page_Photo_ScreenEffects` | Chromatic aberration, film grain, motion blur |
| Filters | `DA_Page_Photo_Filters` | The colour look, and hiding characters |
| Vignette | `DA_Page_Photo_Vignette` | Darkened corners |
| Frames | `DA_Page_Photo_Frames` | Aspect ratio bars |
| Logo | `DA_Page_Photo_Logo` | Your logo, and where it sits |

Every tab holds exactly three rows. The rows are `S_SettingRow` entries, the same structure the options menu uses, so a row is a slider, a toggle or a selector depending on its `Row Widget Class`. That is covered in [Add or change a menu page](../ui/menu_pages_and_options.md).

---

## Every row that ships

| Tab | Row | What it does | Range | Default |
|---|---|---|---|---|
| Camera | `photo.camera.mode` | Keep the gameplay camera where it was, or fly a free one | `Default`, `Custom` | `Custom` |
| Camera | `photo.camera.roll` | Tilts the picture, in degrees | -45 to 45 | 0 |
| Camera | `photo.camera.fov` | Field of view, in degrees | 20 to 120 | 80 |
| Depth of Field | `photo.dof.enabled` | Turns the blur on | off, on | on |
| Depth of Field | `photo.dof.distance` | Sets which distance stays sharp | 0 to 100 | 50 |
| Depth of Field | `photo.dof.intensity` | How strong the blur gets | 0 to 100 | 67 |
| Display | `photo.display.brightness` | Lifts or lowers the whole image | 0 to 100 | 50 |
| Display | `photo.display.sharpness` | Edge sharpening | 0 to 100 | 50 |
| Display | `photo.display.saturation` | Colour strength | 0 to 100 | 50 |
| Screen Effects | `photo.fx.chromatic` | Chromatic aberration | 0 to 100 | 0 |
| Screen Effects | `photo.fx.grain` | Film grain | 0 to 100 | 0 |
| Screen Effects | `photo.fx.motionblur` | Motion blur | 0 to 100 | 0 |
| Filters | `photo.filter.index` | The colour look | `None`, `Noir`, `Sepia`, `Vintage`, `Cold`, `Warm` | `None` |
| Filters | `photo.filter.intensity` | How much of the look is mixed in | 0 to 100 | 100 |
| Filters | `photo.filter.hidechars` | Takes characters out of the shot | `None`, `Player`, `All` | `None` |
| Vignette | `photo.vignette.enabled` | Turns the vignette on | off, on | off |
| Vignette | `photo.vignette.size` | How far the darkening reaches in | 0 to 100 | 23 |
| Vignette | `photo.vignette.intensity` | How dark it gets | 0 to 100 | 100 |
| Frames | `photo.frame.index` | Aspect ratio bars | `None`, `2.39 Ratio`, `1.85 Ratio`, `4:3`, `1:1` | `None` |
| Frames | `photo.frame.colour` | Bar colour | `Black`, `White` | `Black` |
| Frames | `photo.frame.intensity` | Bar opacity | 0 to 10 | 10 |
| Logo | `photo.logo.enabled` | Turns the logo on | off, on | off |
| Logo | `photo.logo.corner` | Which corner it sits in | `Bottom Right`, `Bottom Left`, `Top Right`, `Top Left` | `Bottom Right` |
| Logo | `photo.logo.size` | How big it is | 0 to 100 | 50 |

Four aspect ratios ship, plus the `None` entry that turns the bars off. Five colour looks ship, plus `None`. So a selector that reads "5 of 6" on screen is showing you the off entry as one of its choices.

`photo.frame.intensity` is the odd one. It runs 0 to 10, not 0 to 100 like the other opacity sliders. If you copy that row to build another one, copy `Max Value` with it.

!!! warning
    `Logo Texture` on `BP_PlayerPhotoModeManager` ships empty. Until you set it, the Logo tab still works, the toggle still flips, and nothing appears in the picture. Set your own texture there before you decide the tab is broken. See [Put your own logo, frames and filters in photo mode](photo_mode_branding.md).

---

## Keys and mouse while the panel is open

The bar along the bottom lists seven hints. Five of them are keys, and each one is also a button you can click.

| Hint | Key | What it does |
|---|---|---|
| GRID | `G` | Shows and hides a rule of thirds grid |
| HIDE MENU | `H` | Hides the panel so you can judge the frame clean |
| RESET TAB | `R` | Puts the three rows of the current tab back to their defaults |
| PHOTO | `Enter` | Takes the shot |
| CLOSE | `Esc` | Leaves photo mode |

The other two, ROTATE and ZOOM, are on the mouse: move it to aim the camera, scroll to change the field of view.

Flying the camera uses the normal Input Actions, so they are all rebindable in `IMC_Default`:

- `IA_Move` moves the camera on the ground plane
- `IA_Photo_Elevate` moves it up and down, on `Space` and `Left Ctrl`
- `IA_Photo_Roll` tilts it, on `E` and `A`

Tabs and rows also work with the mouse. Click a tab to open it, click a row to select it, then drag a slider or pick a side of a selector.

---

## What the game does while you shoot

- Time is not paused, it is slowed to `Photo Time Dilation`, which ships at `0.0001`. The world still ticks, so the camera keeps moving while everything in the scene stands still.
- Anything simulating physics nearby is frozen and its velocities are stored, then given back when you leave. Ragdolls are settled first so a body does not twitch during the shot.
- Every game widget is hidden and restored on exit, so the HUD is never in the picture.
- Characters are only hidden if you ask for it, with `photo.filter.hidechars`.

---

## Taking the shot, and what is in it

`Take Photo` hides the panel for the frame, then asks the engine for a high resolution screenshot. `Photo Resolution Scale` ships at `2`, so the file comes out at twice your screen resolution and lands in your project's `Saved/Screenshots` folder. The shutter sound is `Content/TheLastTemplate/Audios/Widgets/SC_PhotoTake`.

The bars, the vignette, the colour look and the logo are not widgets. They are drawn by two post process materials, `M_PhotoModeGrade` and `M_PhotoModeLens`, which means they are part of the render and they end up in the file.

When you leave, `Save Photo Settings` writes every value back into `SG_MenuSettings`, the same save the options menu uses. A player who set up a look keeps it the next time they open photo mode, and after a restart.

---

## Camera limits you can retune

These are plain fields on `BP_PlayerPhotoModeManager`, in the `Details` panel of the component on `BP_PlayerCharacter`. They change how the camera feels, not what the picture looks like.

| Field | Ships at | What it does |
|---|---|---|
| `Max Distance From Player` | `600` | How far the camera may travel from the player, in centimetres |
| `Move Speed` | `350` | Fly speed, in centimetres per second |
| `Move Smoothing` | `12` | Higher means the camera reaches the input faster |
| `Look Sensitivity` | `1` | Mouse look multiplier |
| `Look Smoothing` | `18` | Higher means less lag on the aim |
| `Roll Speed` | `60` | Degrees per second when tilting |
| `Zoom Speed` | `4` | Field of view change per wheel notch |
| `Zoom Smoothing` | `10` | Higher means the zoom settles faster |
| `Min FOV` | `20` | Lower bound of the zoom |
| `Max FOV` | `120` | Upper bound of the zoom |
| `Photo Time Dilation` | `0.0001` | World speed while photo mode is open |
| `Block Camera On Collision` | `true` | Stops the camera going through walls |
| `Photo Resolution Scale` | `2` | Screenshot size, as a multiple of your screen |

`Min FOV` and `Max FOV` match the range on `photo.camera.fov`. If you widen the slider in the Data Asset, widen these two as well, or the zoom clamps before the slider does.

---

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
