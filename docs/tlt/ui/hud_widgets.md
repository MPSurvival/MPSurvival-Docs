# Change what the HUD shows

Everything the player sees during play is a small widget of its own, owned by the system that needs it. There is no single big HUD widget. Changing the ammo counter never touches the health ring, and deleting one element never breaks the rest.

This page says which file to open for each element on screen, and which fields on it are yours. For the wider picture of who creates what, see [How the HUD, menus and saving fit together](screens_overview.md).

The widgets live in `Content/TheLastTemplate/Blueprints/Widgets/`. The materials they draw with live in `Content/TheLastTemplate/Materials/Widgets/`.

| On screen | Widget | Where you change it |
|---|---|---|
| Health ring and stamina bar | `BP_LifeCharacterWidget` | `MI_Round_Progess`, `MI_OverlayBackdrop` |
| Ammo counter and pips | `BP_AmmoCounterWidget` | `MI_AmmoPips`, the weapon Data Asset |
| Equipped throwable icon | `BP_ThrowableWidget` | the throwable Data Asset |
| Crosshair | `BP_AimWidget` | one crosshair material per weapon |
| Interaction prompt | `BP_InteractWidget` | the interactable's Details panel |
| Tip in a corner of the level | `BP_TipWidget` | a `BP_TipZone` you place |
| Notifications | `BP_NotificationWidget` | one function call |
| Weapon and item wheel | `BP_RadialWidget` | the weapon Data Asset |
| Damage flash | `BP_HitCharacterWidget` | `T_HitCharacter` |
| Fade to black | `BP_FadeScreenWidget` | its widget animation |
| Death screen | `BP_DeathScreenWidget` | its own fields |

---

## The health ring and the stamina bar

`BP_LifeCharacterWidget` draws three things: the ring, the stamina bar under it, and the dark backdrop behind both.

The ring is one `Image` running `MI_Round_Progess`. The widget writes `Percentage` into it as health changes. Everything about how the ring looks is in that material instance:

| Parameter | What it does |
|---|---|
| `Percentage` | How much of the arc is filled. Written at runtime, so what you set here is only the editor preview. |
| `Thickness` | Width of the ring stroke. |
| `Smoothness` | Softness of the ring edge. |

`Thickness` and `Smoothness` are not linked. If you make the ring much thicker, raise `Smoothness` with it or the edge turns hard.

The backdrop is a second `Image` running `MI_OverlayBackdrop`. It is a rounded rectangle plus a disc, drawn in the material rather than as a texture, so you can move and resize it without new art: `Rect Center X`, `Rect Center Y`, `Rect Half Width`, `Rect Half Height`, `Rect Corner`, `Disc Radius`, `Panel Opacity` and `Softness`.

The maximum values come from the vitals Data Assets, not from the widget:

| Data Asset | `Vital Max Amount` | `Vital Tick Decrementation` |
|---|---|---|
| `DA_Vital_Health` | 100 | 0 |
| `DA_Vital_Stamina` | 200 | 2 |

Health does not decay on its own. Stamina drains by 2 per tick while sprinting. Raising `Vital Max Amount` gives the player more health without any change to the widget, because the ring is drawn from a percentage.

!!! warning
    `DA_Vital_Health` and `DA_Vital_Stamina` also carry a `Vital Icon` and a `Vital Color`. Both ship empty and the HUD does not read either of them. Setting them changes nothing on screen. The colours of the ring and the bar are in `MI_Round_Progess`.

More on vitals in [Health, stamina and new vitals](../damage/health_stamina_and_new_vitals.md).

---

## The ammo counter

`BP_AmmoCounterWidget` appears when you hold a gun. It has three parts: the weapon icon, the current and reserve numbers, and the pip strip.

The pip strip is a single `Image` running `MI_AmmoPips`. One pip is one round in the clip, so the length of the strip follows `Clip Size`. The count is written by the widget every time ammo changes, so `Max Pips` and `Pip Count` on the material instance only affect the editor preview. The two parameters worth touching are `Pip Fill` and `Softness`, which control how a pip is drawn and how soft its edge is.

The weapon icon comes from the weapon Data Asset, not from the widget:

| Field on the weapon Data Asset | What it does |
|---|---|
| `Weapon Icon` | The texture shown next to the numbers. |
| `Weapon Icon Scale` | Its size, as a 2D vector. Each weapon can have a different one. |
| `Clip Size` | How many pips are drawn. |

The throwable equivalent is `BP_ThrowableWidget`, which shows the throwable you have equipped. It reads `Icon` and `Throwable Icon Scale` from the throwable Data Asset, the same way. See [Add your own throwable](../throwables/add_your_own_throwable.md).

---

## The crosshair

`BP_AimWidget` draws the crosshair while you aim. Each gun brings its own crosshair, so a shotgun and a pistol never look the same.

Two crosshair materials ship, with three instances made from them:

| Material instance | Built from | Shape | Used by |
|---|---|---|---|
| `MI_CrosshairPistol` | `M_CrosshairNormal` | Four ticks | `DA_Pistol_01`, `DA_Pistol_02` |
| `MI_CrosshairShotgun` | `M_CrosshairCircle` | A ring | `DA_Shotgun_01`, `DA_Shotgun_02` |
| `MI_CrosshairRifle` | `M_CrosshairNormal` | Four ticks, own length | nothing yet |

`MI_CrosshairRifle` is there for you. No shipped weapon points at it, because the template ships two pistols and two shotguns.

To give a gun a different crosshair, set `Crosshair Material` on its Data Asset. `Crosshair Display Delay After Aim` on the same asset is how long the crosshair waits before appearing once the player starts aiming.

To change how a crosshair looks, open its instance. `M_CrosshairNormal` exposes `Color`, `Length`, `Thickness` and `Softness`. `M_CrosshairCircle` exposes `Radius`, `Thickness` and `Softness`.

Both materials also have an `Offset` parameter, which the widget writes every frame so the crosshair opens as the weapon's spread grows and closes as it settles. The spread numbers themselves are on the weapon Data Asset: `Ammo Spread`, `Max Ammo Spread` and `Spread Recovery Speed`. See [Change how a gun feels](../weapons/change_how_a_gun_feels.md).

---

## The interaction prompt

`BP_InteractWidget` is not a screen widget. It sits on a `Widget Component` on the interactable itself, so it floats in the world at the object. That is why you never have to register anything: place the actor and the prompt comes with it.

Everything you change is on the actor, in `BP_InteractableOverlapBase` and its children:

| Field | What it does | Default |
|---|---|---|
| `Interact Text` | The label, when you are using text. | empty |
| `Interact Icon` | The texture, when you are using an icon. | none |
| `Use Icon` | Picks between the two. They are alternatives, never both. | `false` |
| `Error Text` | Shown instead when the action is refused, for example a full backpack. | `Full` |
| `Show Interact Zone Range` | Distance at which the prompt appears. | 300 |
| `Interact Zone Range` | Distance at which the key actually works. | 60 |

Pickups override the two distances to 200 and 100, so a prompt on a pickup shows up closer than one on a door.

The prompt has four states: hidden, far, near and near with an error. Far is inside `Show Interact Zone Range` but outside `Interact Zone Range`. You do not set the state yourself, the actor does it as the player moves.

Pickups can also show a count. `Show Amount` on `BP_Interactable_PickupBase` is `false`, and `BP_Interactable_Pickup_Item` turns it on, so an item pickup carries a count next to its label while `BP_Interactable_GateButton` just reads `Open`. The number is `Amount` on the pickup.

If you are writing your own interactable and want to drive the prompt by hand, `BP_InteractWidget` exposes `Set Interact Text`, `Set Interact Icon`, `Set Interact Amount`, `Set Error Text` and `Set Display State`. See [How interaction works](../inventory/how_interaction_works.md).

---

## Tips in the level

A tip is the small line of help that appears near an obstacle, showing a key and a sentence. It is `BP_TipWidget`, and it is spawned by a `BP_TipZone` you drop into the level.

1. Drag `BP_TipZone` from `Content/TheLastTemplate/Blueprints/Environments/Zones/` into the level.
2. Size the box to cover the area where the tip should show.
3. Set `Tip Message` to the sentence you want.
4. Set `Tip Action` to the Input Action whose key you want drawn.
5. Save.

`Tip Action` is an Input Action reference, not a key name, and it ships empty on purpose. The widget resolves the glyph from the player's current binding, so a tip that says Space keeps saying the right thing after the player rebinds. The glyphs come from `DA_KeyPrompts`, set on the widget as `Prompt Set`. See [Key icons and letting players rebind keys](key_prompts_and_rebinding.md).

The tip fades itself out and removes itself when the player leaves the box.

---

## Notifications

A notification is the panel that slides in with a title, a message and an icon. You raise one with a single node from anywhere:

**Show Notification**, in `BFL_NotificationLibrary`.

| Pin | What it is |
|---|---|
| `In Title` | The headline. |
| `In Message` | The body text. Leave it empty for a title only panel. |
| `In Icon` | A `Texture2D`. Leave it empty for no icon. |
| `In Duration` | How long the panel holds before it leaves. |

You do not create or place anything first. The node looks for `BP_NotificationLayerWidget` on screen and makes one if there is none.

Notifications queue. Only one plays at a time, and the next starts when the one before it has finished leaving, so a burst of pickups reads one line at a time instead of stacking.

The pacing and the look are on `BP_NotificationWidget`: `Display Duration`, `Intro Time` and `Outro Time` are its class defaults, and `In Duration` on the call overrides the first of them. The wipe that reveals the panel is `M_NotificationWipe`, driven by `Wipe Phase` and `Wipe Alpha`, with `Reveal` and `Feather` as its material parameters.

---

## The weapon and item wheel

`BP_RadialWidget` is the wheel on the middle mouse button. It has two halves.

The horizontal rows hold your guns. Which row a gun lands in is `Radial Slot Row` on its weapon Data Asset, so two pistols on the same row become one row you scroll through, and a shotgun on another row gets its own line. Add a gun with a new row number and a new line appears, no widget work.

The vertical column holds the medkit and the two throwable slots, primary and secondary. Which one a throwable takes is its `Type` on the throwable Data Asset.

Three slot widgets do the drawing: `BP_Radial_SlotHorizontalWidget` for a filled weapon slot, `BP_Radial_SlotHorizontalEmptyWidget` for an empty one, and `BP_Radial_VerticalWidget` for the column. The highlight colours are on `BP_RadialWidget` itself, as `Selected Background Color` and `Unselected Background Color`.

The wheel slows time while it is open and re-centres the mouse cursor on the screen.

---

## Damage flash, fade to black and the death screen

**The damage flash** is `BP_HitCharacterWidget`. It is one full screen `Image` using `T_HitCharacter`, with two widget animations: one that fades the flash in and out on a hit, and one that keeps pulsing while health stays low. It watches health itself, so it needs nothing from you. To change how being hurt reads, replace `T_HitCharacter` or retime the two animations.

**The fade to black** is `BP_FadeScreenWidget`, a black `Image` plus one animation. The player character calls `Play Fade Animation` on it when the screen has to go dark. Retime the animation and every fade in the game follows.

**The death screen** is `BP_DeathScreenWidget`. Two fields on it are yours:

| Field | What it does |
|---|---|
| `Death Tips` | A list of lines. One is picked at random each death. Add your own, remove the ones you do not want. |
| `Continue Key Icon` | The glyph shown next to the continue prompt. Ships as `T_Key_Enter`. |

Continue loads the most recent save and reopens the level through the loading screen. The death screen is built from the same base widgets as the menus, so it already matches them. See [Add or change a menu page](menu_pages_and_options.md) and [Save and load a game](saving_a_run.md).

---

## Use your own widget instead

There are two ways in, and the first one is almost always the right one.

**Restyle the widget that ships.** Open it, change the layout, the fonts, the textures, the animations. Nothing outside it needs to know. This works for every element on this page.

**Replace it with your own class.** Make your widget a child of the one it replaces, so the code that holds it still finds what it expects, then build your own layout inside the child. The play HUD widgets are created in `Initialize Widgets` on `BP_PlayerCharacter`; swap the class on the Create Widget node there.

To add something new to the HUD, build your widget, then add a Create Widget and an Add to Viewport in `Initialize Widgets` next to the others. Give it a higher Z order than the elements it should sit above.

---

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
