# Change movement speeds and gaits

The player has four **gaits**: walking, running, crouching and narrow passage. A gait is a named set of movement numbers, and all four of them live in one field on the player character.

- The field: `Locomotion Gaits Settings`, on `Content/TheLastTemplate/Blueprints/PlayerCharacter/BP_PlayerCharacter`, in the Details panel under **Settings > Movements**
- The names: `Content/TheLastTemplate/Blueprints/Enumerations/Locomotion/E_LocomotionGaits`
- The numbers inside one gait: `Content/TheLastTemplate/Blueprints/Structures/Locomotion/S_LocomotionGaitSettings`

Changing how fast the player moves means changing a number in that field. You do not edit the Character Movement component: the character writes over it every time the gait changes.

---

## The four gaits

| Gait | When the player is in it | Shipped `Max Walk Speed` |
|---|---|---|
| `Walking` | Default, nothing held | 190 |
| `Running` | While `IA_Sprint` is held | 400 |
| `Crouching` | While crouched | 120 |
| `NarrowPassage` | Inside a `BP_NarrowPassage_Base` volume | 80 |

The keys of the map are the four values of `E_LocomotionGaits`. The sprint and crouch keys themselves are in [Change the controls](../start/change_the_controls.md). The narrow passage volume is in [Flashlight, listen mode, backpack and narrow passages](player_toolkit.md).

---

## Change the sprint speed

1. Open `BP_PlayerCharacter`.
2. Click **Class Defaults**.
3. In the Details panel, open `Locomotion Gaits Settings` under **Settings > Movements**.
4. Expand the `Running` row.
5. Set `Max Walk Speed` to the value you want.
6. Compile and save.

---

## What one gait row holds

| Field | What it does |
|---|---|
| `Max Walk Speed` | Top speed on the ground |
| `Max Acceleration` | How fast the player reaches that speed |
| `Breaking Acceleration` | How hard the player slows down when the stick is released |
| `Braking Friction Factor` | Multiplier applied to the ground friction while stopping |
| `Use Separate Braking Friction` | Use the value below instead of the surface friction |
| `Braking Friction` | Only read when the box above is ticked |

---

## Crouching

The crouch speed is the `Max Walk Speed` of the `Crouching` row, nothing else. `Max Walk Speed Crouched` on the Character Movement component looks like the right place and is not: the character overwrites it with the current gait speed on every refresh, so the `300` it ships with is never used.

Two engine fields on the Character Movement component do matter for crouching:

| Field | Shipped | What it does |
|---|---|---|
| `Can Crouch` | on | Without it, the crouch input does nothing at all |
| `Crouched Half Height` | 60 | The capsule shrinks to this from its standing 88 |

The standing capsule is 88 half height and 30 radius. If you make the crouched capsule shorter, check that the player still fits under the props you expect them to crouch through, and check that they can stand up again where you expect.

---

## Add a fifth gait

Say you want a slow limp gait for a wounded player.

1. Open `E_LocomotionGaits` and add an enumerator. Give it a display name, for example `Limping`.
2. Open `BP_PlayerCharacter`, **Class Defaults**, `Locomotion Gaits Settings`. Add a row, pick `Limping` as the key, and fill in the six numbers.
3. Call `Update Locomotion Gait` on the player with `Limping` when you want it, and with `Walking` when it ends.
4. Compile and save.

!!! warning "A gait with no row in the map stops the player dead"
    The map is the only source of movement numbers. A gait with no row reads back as a row of zeros, so the next speed refresh writes a walk speed of `0`. The player then cannot move and nothing is printed. Add the enumerator and the row in the same pass.

---

## Give another character its own speeds

`BP_NPCCharacter` and `BP_ZombieCharacter` each carry their own `Locomotion Gaits Settings`. Edit theirs the same way, in their own Class Defaults.

| Number | NPC | Zombie |
|---|---|---|
| `Walking` speed | 190 | 100 |
| `Walking` acceleration | 400 | 400 |
| `Running` speed | 400 | 400 |
| `Crouching` speed | 140 | 140 |

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
