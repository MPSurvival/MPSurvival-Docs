# Fall damage, death and the ragdoll

A character that lands hard loses health. A character whose health reaches zero drops to a ragdoll and stays on the ground. Both are already wired on the characters that ship.

---

## Where fall damage lives

One component: `Content/TheLastTemplate/Blueprints/ActorComponents/BP_FallDamageComponent`.

Three characters carry it: `BP_PlayerCharacter`, `BP_NPCCharacter` and `BP_ZombieCharacter`.

Add it to any `Character` and it works. It binds to the character's own landing event, so there is nothing to connect. It reads the fall height from the speed the character hits the ground at and from the world gravity, not from where the fall started, so it is correct however the character got into the air.

The damage itself goes out as a normal `Apply Damage` on the owner. That means fall damage passes through `BP_VitalsSystem` like every other hit, and `Invulnerable` and `Damage Taken Multiplier` apply to it. See [How a character takes damage](how_damage_works.md).

---

## The fields, and what ships

Select `BP_FallDamageComponent` in the `Components` tree of a character to see these in the `Details` panel. The values below are the ones the component itself ships with. Heights are in centimetres, like everything else in Unreal.

| Field | Shipped | What it does |
|---|---|---|
| `Fall Damage Enabled` | `true` | Uncheck to switch fall damage off for that character. |
| `Safe Fall Height` | `300` | Fall this far or less and nothing happens. |
| `Lethal Fall Height` | `900` | At this height and beyond you take `Max Fall Damage`. |
| `Min Fall Damage` | `8` | The damage a fall costs the moment it passes `Safe Fall Height`. |
| `Max Fall Damage` | `150` | The damage at `Lethal Fall Height` and above. |
| `Damage Exponent` | `2` | The shape of the curve between the two heights. |

Two more fields are on the component but are filled in at play time, not authored: `Last Fall Damage` and `Last Fall Height`. They hold the last landing, and they are useful when you react to a fall.

### What the shipped numbers give you

Below three metres, nothing. Above nine metres, 150 damage. A character starts with 100 health, so nine metres is always fatal, which is what the name of the field promises.

Between the two, `Damage Exponent` decides. At `1` the damage climbs in a straight line. At `2`, the shipped value, it stays low for the first half of the range and then rises fast, so a medium drop is survivable and a bad one is not:

| Fall | Damage |
|---|---|
| 3 m or less | none |
| 4.5 m | about 17 |
| 6 m | about 44 |
| 7.5 m | about 88 |
| 9 m or more | 150 |
## Turn fall damage off for one character

Uncheck `Fall Damage Enabled` on that character's component. Nothing else has to change, and the component can stay where it is.

If you are building a character that must never take fall damage at all, simply do not add the component. `BP_Dummy` is set up that way.

---

## React to a fall from your own Blueprint

The component has an event dispatcher, `On Fall Damage`, which fires with the damage and the fall height every time a landing hurts. Nothing in the template listens to it. It is there for you.

1. In your character's `Event Graph`, drag off `BP_FallDamageComponent`.
2. Pick `Assign On Fall Damage`, or `Bind Event to On Fall Damage` if you already have an event to use.
3. Do what you want with the two values: a grunt, a camera shake, a limp, a broken leg state.

---

## What death does by default

Health reaching zero is `BP_VitalsSystem`'s business. It raises its `On Death` dispatcher, and each character decides for itself what that means.

An AI, `BP_NPCCharacter` or `BP_ZombieCharacter`, stops its montage and hands the skeletal mesh over to physics. The body falls, keeps its momentum, and stays where it lands. Nothing destroys it and nothing cleans it up. If you want corpses to disappear, add a `Set Life Span` after the ragdoll starts.

The player disables input, plays the same ragdoll, and puts `BP_DeathScreenWidget` on screen. The widget class it uses is a field on the player, `BP Death Character Widget`, so you can swap in your own screen without opening a graph. The screen has two fields of its own in `Class Defaults`: `Death Tips` for the line of text drawn at random, and `Continue Key Icon` for the prompt.

---

## `PlayDeathRagdoll`, one per character class

`PlayDeathRagdoll` is a function on the interface `Content/TheLastTemplate/Blueprints/Interfaces/BPI_AICombatant`. `BP_PlayerCharacter`, `BP_NPCCharacter` and `BP_ZombieCharacter` each implement their own version and call it from their own death handler.

That is deliberate. A player death has a screen to bring up, an AI death does not, and a character of yours will have its own needs. The interface only fixes the name, so anything that implements it can be asked to die the same way.

To give your own character a ragdoll death:

1. Open your Character Blueprint and add the `BPI_AICombatant` interface in `Class Settings`.
2. Implement `PlayDeathRagdoll`.
3. In it, call `Prepare for ragdoll` on `BP_HitReactionManager`, then stop the montage, then start simulating physics on the mesh.
4. Bind the `On Death` dispatcher of `BP_VitalsSystem` in `Begin Play` and call `PlayDeathRagdoll` from it.

---

## The physics asset the ragdoll uses

One asset for every character: `Content/TheLastTemplate/Meshes/Characters/PA_HumanoidCharacter`. It is set on `SK_PlayerCharacter`, `SK_NPC_01`, `SK_NPC_02` and `SK_Zombie_01`.

So the way a body falls, folds and settles is tuned in one place, and the change lands on the player, the NPCs and the zombie at the same time. Open it, change the bodies, the constraint limits or the masses, and save.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
