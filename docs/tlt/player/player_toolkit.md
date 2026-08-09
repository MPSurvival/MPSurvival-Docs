# Flashlight, listen mode, backpack and narrow passages

Four small systems that hang off the player. None of them is built inside the character graph: the light is its own actor, listen mode is a post process material, the backpack is an actor plus one flag on the player, and a narrow passage is a placeable Blueprint. So you retune all four without opening `BP_PlayerCharacter`.

The keys they use are listed with everything else in [Change the controls](../start/change_the_controls.md).

---

## The flashlight

The light is an actor, `BP_PlayerLight`, at `Content/TheLastTemplate/Blueprints/PlayerCharacter/Misc/BP_PlayerLight`. The player carries it on a scene component called `LightSocket`, through a child actor component called `BP_SocketLightChild`.

Inside the actor there are three components that matter:

| Component | What it is |
|---|---|
| `Light` | Static mesh, `SM_Flashlight`. The torch itself. |
| `SpotLight` | The beam. |
| `PointLight` | The soft light around the lamp itself. |

And two fields:

| Field | What it does | Ships as |
|---|---|---|
| `IES Texture` | The list of light profiles the change-light key steps through. | 6 entries |
| `Current Light Type Index` | Which entry of that list the light starts on. | `0` |

The six shipped profiles all come from `/Engine/EngineLightProfiles/`: `180Degree_IES`, `90Degree_IES`, `Complex_IES`, `Narrow10000`, `Narrow1000_IES`, `NarrowComplex_IES`. They run from a wide wash to a tight beam, which is what the change-light key is for.

The player does not start with a light. `BP_Interactable_Light`, in `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/Misc/`, is the pickup that gives it. It is a plain child of the pickup base: `SM_Flashlight` as its mesh, `Light` as its prompt text.

### Change the light types

1. Open `BP_PlayerLight` and go to **Class Defaults**.
2. Under `IES Texture`, add, remove or replace entries. Any **Texture Light Profile** asset works, including your own imported `.ies` files.
3. Set `Current Light Type Index` to the entry you want the player to start on.
4. Compile and save.

Nothing else changes. The change-light key reads the array at runtime.
## Listen mode

Listen mode drains the colour out of the world and draws characters as glowing silhouettes through walls. It only runs **while you are crouched**. Pressing the key standing up does nothing, and that is deliberate: it is a thing you do when you have stopped moving, not a way to see through walls all the time. Crouching is covered in [Change movement speeds and gaits](movement_speeds_and_gaits.md).

The effect is one post process material:

- `Content/TheLastTemplate/Materials/Effects/PostProcess/M_TLTPostProcess`
- and the instance you actually edit, `MI_TLTPostProcess`

The instance is a blendable on the post process component of `BP_VisualEffects`. Its strength is not a variable on the material: the player writes into the material parameter collection `MPC_PlayerMaterials`, which has a single scalar called `ListenMode`, and the material reads it. That is why the fade in and out is smooth and why nothing has to be spawned when the key is pressed.

!!! warning
    Listen mode is invisible in a level that has no `BP_VisualEffects` actor in it. That actor is what carries `MI_TLTPostProcess`, and it is the only thing in the project that does. `L_ShowcaseMap` and `L_LocomotionTesting` each have one. If you build a new map and the key seems dead, that is the reason.

### The values that shape it

Open `MI_TLTPostProcess` and you get these. They are the whole effect.

| Parameter | Ships as | What it does |
|---|---|---|
| `StencilValue` | `1` | The custom depth stencil value a mesh must carry to be revealed. |
| `BlockStencil` | `2` | The custom depth stencil value that marks a surface as blocking a silhouette behind it. |
| `NearDistance` | `300` | Under this distance in centimetres a silhouette is at full strength. |
| `FarDistance` | `3500` | The distance at which a silhouette has faded away completely. |
| `BlurRadius` | `15` | Width of the search that builds the outline. Bigger is softer. |
| `GlowBoost` | `0.2` | How bright the silhouettes are. |
| `Desaturation` | `0.9` | How much colour is pulled out of the world while the mode is on. |
| `VignetteStrength` | `1.7` | How hard the corners of the screen go dark. |
| `FogDistance` | `10000` | How far the fog layer reaches. |
| `FogOpacity` | `0.1` | How strong that fog is. |
| `SilColor` | white | Colour of the silhouettes. |
| `TintColor` | grey `0.7708` | Tint laid over the drained world. |
| `FogColor` | white | Colour of the fog layer. |

### Set how far it reaches

`FarDistance` is the range, in centimetres. Shipped at `3500`, so 35 metres. `NearDistance` at `300` is where the fade starts, so anything inside 3 metres is drawn at full strength. Raise `FarDistance` for a longer reach; keep the gap between the two wide or the fade turns into a hard edge.

### Make your own actor show through walls

The material looks for meshes in the custom depth buffer, so this is two engine checkboxes and no code:

1. Select the mesh component on your actor.
2. In the Details panel, under **Rendering**, tick `Render CustomDepth Pass`.
3. Set `CustomDepth Stencil Value` to `1`, matching `StencilValue` on the material.

To hide something from listen mode, leave `Render CustomDepth Pass` off. That is all it takes, and it is why props do not glow.

Three shipped classes drive that flag from their own graph rather than leaving it on all the time: `BP_NPCCharacter`, `BP_ZombieCharacter` and `BP_Dummy`. Copy that if you want a target that only shows up sometimes.

---

## The backpack

The backpack is a gate, not a container. Until the player has it, the inventory screen does not open, and crafting and the collectibles list live on that screen, so they come with it.

- The pickup is `BP_Interactable_Backpack`, in `.../Interactable/Pickup/Misc/`. Mesh `SM_Backpack`, prompt text `Backpack`.
- What the player then wears is `BP_PlayerBackpack`, in `.../PlayerCharacter/Misc/`. It holds one component, `BackpackSkeletalMesh`, using `SK_Backpack_01`.
- The flag is `Is Backpack Equipped` on the player. `Has Backpack` is what goes into the save game, so a player who found the bag still has it after a reload.

Five assets read `Is Backpack Equipped`, which is the full list of what the bag unlocks:

| Asset | Why it asks |
|---|---|
| `BP_PlayerInventoryManager` | The inventory screen, crafting and collectibles. |
| `BP_PlayerWeaponManager` | How weapons are carried. |
| `BP_Interactable_Pickup_Item` | Picking items up at all. |
| `BP_Interactable_Workbench` | Using the workbench. |
| `BP_Interactable_Backpack` | The bag pickup itself, so it knows whether the player already has one. |

There is no field that starts the player with a bag. If you want one from the first second, either drop a `BP_Interactable_Backpack` where the player spawns, or call `Equip Backpack` on the player character at Begin Play.

`Secondary Backpack Holster Socket` on the player is set to `SecundarySocketColdreBackPack`. That is where a second weapon rides once the bag is on, and the weapon side of it is two fields on the weapon Data Asset: `Use Unholster Backpack Socket` and `Unholster Backpack Socket`. See [Weapon overlays and animation layers](weapon_overlays_and_layers.md) for how the held weapon is presented.

---

## Narrow passages

A narrow passage is the squeeze-between-two-walls move. It is a placeable actor, `BP_NarrowPassage_Base`, in `Content/TheLastTemplate/Blueprints/Environments/NarrowPassage/`, with one ready-made variant, `BP_NarrowPassage_01`.

The base is two meshes, four boxes and an arrow:

| Component | What it is |
|---|---|
| `MeshLeft`, `MeshRight` | The two walls. They ship as plain cubes wearing `MI_NarrowPassage` so you can see the shape while you place it. |
| `ActiveNarrowPassage` | The box you have to be inside for the mode to hold. |
| `DeactivateNarrowPassage` | The box that drops you back out of it. |
| `ActivateActionA`, `ActivateActionB` | The two entry boxes, one at each end. |
| `ActivateActions` | An arrow that orients those two ends. |

And four fields:

| Field | Ships as | What it does |
|---|---|---|
| `Activate Align Control` | off | Turn it on and the passage takes the camera and turns it to face along itself. |
| `Delay Between Enter` | `3` | Seconds before the same passage will take you again after you left it. |
| `Delay Between Enter Exit` | `0.1` | The second half of the same guard, on the entering-then-leaving direction. |
| `Is Pin` | on | Shipped ticked on the base, and `BP_NarrowPassage_01` does not change it. |

While you are inside, the character switches to the `NarrowPassage` locomotion gait, and the animation comes from `BS_HumanoidNarrowPassage`, in `Animations/BlendSpaces/`, which blends `AS_NarrowPassage_Forward` and `AS_NarrowPassage_Backward` from `Animations/AnimSequences/Characters/NarrowPassage/`.

### Make your own

1. Right click `BP_NarrowPassage_Base`, then **Create Child Blueprint Class**.
2. Set your own meshes on `MeshLeft` and `MeshRight`.
3. Move `ActivateActionA` and `ActivateActionB` to the two mouths of your gap, and point `ActivateActions` along it.
4. Resize `ActiveNarrowPassage` to cover the gap and `DeactivateNarrowPassage` to sit just past the ends.

You do not touch the graph. `BP_NarrowPassage_01` is exactly this and adds no fields of its own.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
