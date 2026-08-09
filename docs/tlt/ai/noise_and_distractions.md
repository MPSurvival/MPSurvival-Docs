# Noise, and how to use it

A noise in this template is one report, made at a point in the world, with a range in centimetres. An AI close enough to it walks over and looks. It never turns the AI hostile on its own, and that is exactly what makes a thrown can useful.

This page is the emitting side: what makes noise, how far each one carries, and how to add your own. The listening side is in [How the AI sees, hears and reacts](how_the_ai_sees_and_hears.md).

---

## The range belongs to the noise, not to the AI

Every noise carries its own range. An AI hears it only if it is inside that range **and** inside the `Hearing Range` on its `AIPerception` component, which ships at 8000.

So `Hearing Range` is a ceiling. The number you actually feel in play is the one on the emitter, and raising the ceiling never makes a quiet noise travel further. All tuning happens on the thing making the sound.

Where a field next to a range is called `Loudness` or `Amount`, it multiplies that range. A `Max Range` of 2500 with a `Loudness` of 1.5 is heard out to 3750.

---

## What makes noise in the shipped project

| Source | Where its range lives | Ships at |
|---|---|---|
| A walking footstep | `Walk Noise Range` on `AN_FootstepSound` | 450 |
| A running footstep | `Run Noise Range` on `AN_FootstepSound` | 900 |
| A box you place in the level | `Max Range` on `BP_NoiseZone` | 1600 |
| A throwable landing | `Radius` on its noise payload row | 2500, and 5000 for the molotov |
| A gunshot | Reported by `BP_WeaponManager` when the weapon fires | see below |

That is the whole list. Explosions, fire, breaking glass, hit reactions and doors are sound only, and an AI never hears them. A `BP_ExplosionZone` you drag into a level makes no noise at all. If a fight is audible from far away in your game, it is the gunshots doing it.

---

## Footsteps

`AN_FootstepSound` turns a step into a noise. It also picks the surface sound, so it is the same notify that gives you gravel and metal underfoot. It sits on the animations as `AN_FootstepSound_L` and `AN_FootstepSound_R`, one per foot socket.

Open `Content/TheLastTemplate/Animations/AnimNotifies/Character/AN_FootstepSound` and there are two numbers to change:

| Field | What it does | Ships at |
|---|---|---|
| `Walk Noise Range` | Range reported by a walking step | 450 |
| `Run Noise Range` | Range reported by a running step | 900 |

Change them, save, done. It covers everyone at once: the player, the hunters and the infected all run `ABP_HumanoidCharacter` on the same `AS_Humanoid_*` animation set, so there is a single place to edit and no per character copy to keep in sync.

The same notify is on `AM_Character_Land` and on the narrow passage animations, so landing after a drop and squeezing through a gap report a noise as well.

For the surface half of that notify, see [How surfaces work](../look/how_surfaces_work.md).

---

## Gunshots

Firing reports a noise from `BP_WeaponManager`. There is no noise field on `BP_WeaponDataAsset`, so a gun's noise range is not something you type in per weapon.

What you change from the outside is the upgrade. `E_WeaponUpgradeStat` has a `Noise Range` entry, and the shipped `DA_Upgrade_Silencer` is built on it:

| Field on the modifier | Value |
|---|---|
| `Stat` | `Noise Range` |
| `Operation` | `Multiply` |
| `Value` | `0.35` |

The same upgrade level swaps the shot for `SC_SilencerShoot` and attaches `SM_Attachment_Silencer`. Copy that modifier row into an upgrade of your own and you have a second quiet weapon. See [Add a weapon upgrade](../weapons/add_a_weapon_upgrade.md).

---

## A noise you can throw

Every throwable makes a noise where it lands. It is a row in `Impact Payloads` on the throwable's Data Asset with `Payload Class` set to `BP_ThrowablePayload_Noise`. On that row, `Radius` is the noise range and `Amount` is its loudness.

| Data Asset | `Radius` | `Amount` | What else is on the same landing |
|---|---|---|---|
| `DA_Throwble_Can` | 2500 | 1.5 | nothing |
| `DA_Throwble_Bottle` | 2500 | 1.5 | 10 impact damage |
| `DA_Throwble_Brick` | 2500 | 1.5 | 25 impact damage |
| `DA_Throwble_Bomb` | 2500 | 1.5 | an explosion |
| `DA_Throwble_NailBomb` | 2500 | 1.5 | impact damage, a bed of nails |
| `DA_Throwble_Molotov` | 5000 | 1.5 | impact damage, a fire pool |

The can is the pure decoy: the noise row is its only payload, so it hurts nothing and does nothing but move people. Copy it when you want a second distraction object with its own mesh and its own throw feel, and give it one noise row and no others. See [Add your own throwable](../throwables/add_your_own_throwable.md).

---

## A box that makes noise when someone walks in

`BP_NoiseZone` is a box you drag into the level. When an actor carrying a `BP_FactionComponent` enters it, the box reports a noise and plays its `Sound` so you can hear it fire. It ships with `Loudness` at 1, `Max Range` at 1600 and `SC_PerceptionNoise`.

Use it for a floor of broken glass, a rotten staircase, a tripped alarm. It is listed with the other placeable zones in [Place hazards and traps in your level](../throwables/place_hazards_and_traps.md).

---

## Two rules that decide what a noise does

**Only hostiles react.** A noise reaches an AI only if that AI is hostile to whoever made it. The check lives on the listening side, in `BP_AIBrain`, never on the emitter. One noise can therefore pull an enemy patrol while its own allies standing next to it ignore it, and you never have to tag a sound with who it is meant for. Who counts as hostile comes from the faction assets: see [Factions and who fights who](factions_and_targets.md).

**A noise never hands over a target.** It writes a last known location and puts the AI into its search. The AI walks there and looks around. If you have gone, it gives up and returns to its idle pattern. If you are still standing there, its eyes pick you up and the fight starts from sight.

That split is the whole reason distractions work. Throwing a bottle moves an AI without aggravating it. It also means no sound can start a fight by itself, so when you want one to, place the noise where the walk to it swings the AI's vision cone onto the player.

---

## Making something quieter, or silent

Lowering a range works the same everywhere: a smaller `Walk Noise Range`, a smaller `Radius` on a throwable's noise row, a smaller `Max Range` on a zone.

!!! warning "Zero does not mean silent"
    A range of zero or less means **no limit**, not no noise. Set one to `0` and you get the loudest thing in your game, stopped only by the AI's `Hearing Range` of 8000. Use `1` when you want something to be effectively inaudible.

To take hearing away from a listener instead of quietening the world, `Use Hearing` on the AI Data Asset switches the sense off for every AI using that asset. It ships true on all five presets. An AI with hearing off still sees you and still reacts to being shot.

---

## Picking your own numbers

The shipped values form a ladder that is easy to keep in your head, in metres:

| Noise | Range |
|---|---|
| A walking step | 4.5 |
| A running step | 9 |
| A noise box | 16 |
| A throwable landing | 25 |
| A molotov | 50 |
| The AI's ceiling | 80 |

Stay inside that ladder and the game stays readable to a player: running is clearly worse than walking, and a thrown bottle is clearly worse than either. Two ends of it are wasted effort. Anything above 8000 does nothing, because `Hearing Range` cuts it. Anything under a couple of hundred is heard so rarely that it reads as silence, which is fine if that is what you wanted and confusing if it is not.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
