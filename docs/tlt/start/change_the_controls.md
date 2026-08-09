# Change the controls

The template uses **Enhanced Input**. Almost everything the player presses goes through an **Input Action**, and every Input Action is listed once in a single **Input Mapping Context** called `IMC_Default`. A few keys are read directly instead, and they are listed further down.

That means one asset to open to change a key, and one folder to look at to see most of what the player can do.

- The mapping context: `Content/TheLastTemplate/Inputs/IMC_Default`
- The actions: `Content/TheLastTemplate/Inputs/Inputs/`

---

## The actions that ship

Twenty two Input Actions come with the template.

| Action | What it does | Default key |
|---|---|---|
| `IA_Move` | Walk and run | Z, Q, S, D |
| `IA_Look` | Look around | Mouse |
| `IA_Sprint` | Sprint while held | Left Shift |
| `IA_Crouch` | Crouch | Left Ctrl |
| `IA_Vault` | Vault, mantle, hurdle and climb | Space |
| `IA_Dodge` | Dodge, in melee only | Space |
| `IA_Jump` | Jump | not mapped |
| `IA_Interact` | Use the object you are looking at | E |
| `IA_Aim` | Aim down the weapon | Right mouse button |
| `IA_Reload` | Reload | R |
| `IA_Unequip` | Put everything away | ) |
| `IA_Finisher` | Stealth finisher, from behind | F |
| `IA_ListenMode` | Listen mode, while crouched | A |
| `IA_OpenBackpack` | Open and close the backpack | Tab |
| `IA_ToggleLight` | Turn the flashlight on and off | G |
| `IA_ChangeLightType` | Switch to another light | H |
| `IA_InspectAction1` | First button while inspecting an object | E |
| `IA_InspectAction2` | Second button while inspecting an object | F |
| `IA_PauseMenu` | Open the pause menu | Y |
| `IA_DevMenu` | Open the developer menu | ! |
| `IA_Photo_Elevate` | Move the photo camera up and down | Space and Left Ctrl |
| `IA_Photo_Roll` | Tilt the photo camera | E and A |

All of these defaults live in `IMC_Default`. Change one there and the whole game follows, including the key icons drawn on screen and the rebinding list in the options menu.

Two actions share a key on purpose. `IA_Vault` and `IA_Dodge` are both on Space because they can never be available at the same time: the dodge is only allowed when you are in melee, empty handed. `IA_Interact` and `IA_InspectAction1` share E for the same reason.

`IA_Jump` ships as an asset but has no key. The template gives Space to the traversal system instead, because a survival character that climbs is more useful than one that hops. If you want a real jump back, map `IA_Jump` to a free key.

---

## Change a key

1. Open `Content/TheLastTemplate/Inputs/IMC_Default`.
2. Find the row with the action you want to change.
3. Click the key field and press the new key.
4. Save.

That is the whole job. You do not need to touch the player character, the HUD, or the options menu. The key icons on screen read the mapping at runtime, so a prompt that used to show **E** shows the new key on the next play.

One prompt does not follow: the interaction prompt draws its **E** from a texture with the letter baked into it. Rebind `IA_Interact` and that one still shows an E until you swap the texture. Every other prompt in the game updates on its own. How prompts pick their icon is covered in [Key icons and letting players rebind keys](../ui/key_prompts_and_rebinding.md).

---

## Keys that are not Input Actions

A few inputs are read directly on the player character instead of going through the mapping context:

- the **left mouse button**, which fires the weapon you hold or throws a punch when your hands are empty
- the **middle mouse button** and the **mouse wheel**, which open and turn the radial menu
- a small number of extra keys used for testing

Because these do not go through an Input Action, they do not appear in the rebinding list and the player cannot change them. If you need one of them to be rebindable, make an Input Action for it and swap the node in the player character graph. Nothing else has to change.

---

## Letting the player rebind keys in game

The options menu already has a **Controls** page that lists the rebindable actions and lets the player set their own key. It is built from a Data Asset, so an action you add can be listed there without touching a widget. See [Add or change a menu page](..

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
