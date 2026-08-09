# How a character takes damage

Health, damage and hit reactions are one layer, and it is the same layer on the player, on the AI and on the practice dummy. Three components do the work, and only one of them ever removes a hit point.

| Component | What it owns |
|---|---|
| `BP_VitalsSystem` | The vitals themselves: health, stamina, anything you add. The only thing in the project that subtracts. |
| `BP_HitReactionManager` | The physical reaction. The body bends where it was hit, then recovers. |
| `BP_FallDamageComponent` | Turns a fall into an amount of damage and sends it. |

All three live in `Content/TheLastTemplate/Blueprints/ActorComponents/`.

`BP_VitalsSystem` holds a `Vitals` array, one row per vital, and each row points at a vital Data Asset. Two of those ship: `DA_Vital_Health` and `DA_Vital_Stamina`. See [Change health and stamina, or add a new vital](health_stamina_and_new_vitals.md).

---

## Damage arrives through Unreal's own damage events

You do not need a custom function to hurt something in this template. The standard nodes work:

- `Apply Damage` and `Apply Radial Damage` land on `Event Any Damage`.
- `Apply Point Damage` lands on `Event Point Damage`, the only path that knows which bone was hit.

Both paths end at the same call, `Remove Vital Amount` on `BP_VitalsSystem`.

The two paths are kept apart by a damage type, `Content/TheLastTemplate/Blueprints/Misc/DamageTypes/DT_PointHit`.

`Event Point Damage` does three things `Event Any Damage` does not:

1. it scales the damage by the bone that was hit,
2. it stores the bone, the direction and the location of the hit on the character,
3. it hands the hit to `BP_HitReactionManager`.

!!! warning
    When you call `Apply Point Damage` yourself, set **Damage Type Class** to `DT_PointHit`. With any other damage type both events fire and the character loses health twice, once at full value and once scaled by the bone. Nothing warns you.

Radial damage has no handler of its own. An explosion reaches a character through `Event Any Damage`, so its amount is subtracted as written, with no bone multiplier and no procedural reaction.

---

## Where bone multipliers come from

`Event Point Damage` calls `Get Bone Damage`, in `Content/TheLastTemplate/Blueprints/Functions/BFL_DamageLibrary`. It takes the base damage and the bone name and returns the damage to apply.

| Bone | Multiplier |
|---|---|
| `head`, `neck_01`, `neck_02` | 5 |
| `spine_03`, `spine_04`, `spine_05` | 2 |
| `thigh_l`, `thigh_r`, `calf_l`, `calf_r` | 0.8 |
| `clavicle_l`, `clavicle_r` | 0.7 |
| `upperarm_l`, `upperarm_r`, `lowerarm_l`, `lowerarm_r` | 0.5 |
| `Hand_L`, `Hand_R`, `foot_l`, `foot_lr`, `ball_l`, `ball_r` | 0.4 |
| `spine_01`, `spine_02`, `Pelvis` | 1 |
| any bone that is not listed | 1 |

Those are the Unreal mannequin's bone names. `foot_r` is not among them, so a hit on the right foot is not matched and takes plain damage. A skeleton that names its bones differently falls through the same way, everywhere including the head, so add its bone names to `Get Bone Damage` if you retarget the template onto one.

---

## What one removal actually does

`Remove Vital Amount` takes a vital type and an amount, and in that order it:

- multiplies the amount by `Damage Taken Multiplier` before subtracting from health. It ships at 1. This is the field the damage resistance skill writes, and you can write it yourself, see [Add or change a skill](../progression/add_or_change_a_skill.md).
- subtracts, and clamps the vital at 0.
- fires `On Health Damage`. Bind to that dispatcher for screen effects, audio or anything of your own.

When health reaches 0 the component pauses the vital drain, turns `Invulnerable` on, sets `Is Dead`, and fires `On Death`. What the body does after that belongs to the character class. See [Fall damage, death and the ragdoll](fall_damage_and_death.md).

---

## Invulnerability is one flag for the whole actor

`BP_VitalsSystem` carries a single `Invulnerable` boolean. Not one per vital, not one per body part. Two functions drive it: `Set Invulnerable State` and `Toggle Invulnerable State`.

While it is on the character cannot die. The component switches it on itself the moment a character dies, so a body cannot be killed a second time.

---

## What already has the setup

- `BP_PlayerCharacter`, `BP_NPCCharacter` and `BP_ZombieCharacter` carry all three components and both damage events.
- `BP_Dummy` carries `BP_VitalsSystem` and `BP_HitReactionManager` and no fall damage. It is the ragdoll test target.
- `BP_TrainingDummy` is not a Character and has no vitals at all. It is two meshes on a physics constraint with one field, `Impulse Per Damage`. It moves when you hit it and that is all it does.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
