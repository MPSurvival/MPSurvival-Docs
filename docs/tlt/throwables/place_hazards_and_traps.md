# Place hazards and traps in your level

The same actors a throwable spawns when it lands can be dragged straight into a level. That gives you a fire pool in a corridor, a bed of nails in a doorway, a box that hurts anything walking through it, a box that pulls the AI towards it, and a tripwire rigged to an explosion.

They live in two folders:

- `Content/TheLastTemplate/Blueprints/Environments/Zones/`
- `Content/TheLastTemplate/Blueprints/Environments/Traps/`

---

## The hazard actors at a glance

| Actor | What it does | What sets it off |
|---|---|---|
| `BP_ExplosionZone` | Damage in a radius, physics push, explosion effect and sound | Itself, as soon as it starts |
| `BP_MolotovZone` | A pool of fire that sets actors alight | Itself, then anything entering it |
| `BP_NailsZone` | A patch of shrapnel that damages what stands in it | Anything entering it |
| `BP_DamageZone` | A plain damage box, no effect, no sound | Anything entering it |
| `BP_NoiseZone` | Reports a noise the AI hears and comes to investigate | Anything entering it |
| `BP_Tripwire` | A wire across a gap that sets off an explosion | The first actor to touch the wire |

!!! warning "Every zone ships with its damage numbers at zero"
    `Radius`, `Damage`, `Fire Damage`, `Fire Duration` and `Duration` are all `0` on the class. That is on purpose: the throwables push their own numbers in when they spawn a zone, so the class defaults are never used. A zone you place by hand does nothing at all until you type values into it.

Values the shipped throwables push in, if you want a starting point:

| Zone | Comes from | Numbers used |
|---|---|---|
| `BP_ExplosionZone` | the grenade | `Radius` 500, `Damage` 1000 |
| `BP_MolotovZone` | the molotov | `Radius` 200, `Fire Damage` 5, `Fire Duration` 10 |
| `BP_NailsZone` | the nail bomb | `Radius` 120, `Damage` 3, `Duration` 10 |

---

## Place a zone

1. Drag the actor from the Content Browser into your level.
2. Select it and open the `Details` panel.
3. Fill in the numbers below for that zone. Leaving them at `0` is the one mistake that makes a hazard look broken.
4. Play.

If you want the same hazard many times, make a child Blueprint of the zone, set your numbers as its class defaults, and place that instead.

---

## Explosions: `BP_ExplosionZone`

This one goes off on its own, the moment it starts. Placed in a level, it explodes when the level loads. That is what it is for: something else spawns it where and when the blast should happen. To arm a real trap, use `BP_Tripwire` below, or spawn the zone from your own Blueprint.

It applies its damage to everything inside `Radius`, pushes physics bodies away from the centre, plays `NS_Explosion_Grenade` and `SC_Explosion`, then removes itself when the effect has finished.

| Field | What it does | Ships at |
|---|---|---|
| `Radius` | How far the blast reaches, in centimetres | `0` |
| `Damage` | Damage applied inside the radius | `0` |
| `Impulse Strength` | How hard physics bodies are thrown | `5000` |
| `Impulse Delay` | Short wait before that push, so it lands after the flash | `0.15` |

---

## Fire pools: `BP_MolotovZone`

When it starts, it traces down to find the floor it is standing over, sits itself `Surface Offset` above that surface, lights `NS_Fire_MolotovBurst` and scatters fire spots across `Radius`. Anything that walks into it and carries the `CanBeFire` actor tag starts burning.

The burning itself is not done by the zone. It adds a `BP_FireComponent` to the actor, and that component does the damage and the flames. See [Make an actor catch fire](make_an_actor_catch_fire.md) for the tag and the one mesh setting that silently stops flames from appearing.

| Field | What it does | Ships at |
|---|---|---|
| `Radius` | How wide the pool spreads | `0` |
| `Fire Damage` | Damage the fire does to what it catches | `0` |
| `Fire Duration` | How long the fire lasts | `0` |
| `Snap Down Range` | How far down it looks for a floor | `1500` |
| `Snap Side Range` | How far sideways it looks for a wall or ledge | `150` |
| `Surface Offset` | How far it lifts off the surface it found | `3` |

The three `Snap` fields exist because a thrown bottle lands anywhere. Placing a pool by hand, you can leave them alone as long as there is floor under the actor.

---

## Nail beds: `BP_NailsZone`

Plays `NS_NailBomb_Shrapnel` when it starts and damages anything that enters its box. After `Duration` seconds it removes itself, so a hand placed one is a timed patch, not a permanent floor trap. If you want it to stay, give `Duration` a large number.

| Field | What it does | Ships at |
|---|---|---|
| `Radius` | How wide the patch is | `0` |
| `Damage` | Damage dealt to what enters it | `0` |
| `Duration` | Seconds before the zone removes itself | `0` |

---

## A plain damage box: `BP_DamageZone`

No effect, no sound, no lifetime. It is built on the engine's `Trigger Box`, so you resize it with the brush handles like any other trigger volume, and it hurts whatever walks in. Use it for spikes you modelled yourself, for a pit floor, or for a scripted area you want to be dangerous without dressing it.

| Field | What it does | Ships at |
|---|---|---|
| `Damage` | Damage applied on entry | `0` |

Damage goes through the normal path, so hit reactions, vitals and death all behave as usual. See [How damage works](../damage/how_damage_works.md).

---

## A noise box: `BP_NoiseZone`

This one hurts nobody. When something enters it, it calls `Make Noise` with `Loudness` and `Max Range` and names the player as the source, so nearby AI hear it and come looking. It also plays `Sound` so you can hear that it fired.

| Field | What it does | Ships at |
|---|---|---|
| `Loudness` | How loud the reported noise is | `1` |
| `Max Range` | How far the noise carries, in centimetres | `1600` |
| `Sound` | Sound played when it fires | `SC_PerceptionNoise` |

Because the player is named as the source, the AI reacts as if the player had made the sound and walk to it. That is what makes it useful for pulling patrols off a route. For how hearing is tuned on the AI side, see [Noise and distractions](../ai/noise_and_distractions.md).

---

## The tripwire: `BP_Tripwire`

`BP_Tripwire` is a spline. Drag it in, then drag its spline points to stretch the wire across a doorway or between two trees. It builds itself as you move the points: an anchor mesh at each end, `Cable Mesh` repeated along the spline, and a line of invisible box triggers laid out every `Trigger Step` with a thickness of `Trigger Thickness`.

The cable mesh has no collision. Those two numbers are the real hitbox, so if a character can step over your wire without setting it off, raise `Trigger Thickness` or lower `Trigger Step`.

The first actor to touch a box spawns a `BP_ExplosionZone` with your `Explosion Radius` and `Explosion Damage`, and flips `Triggered` so the wire can only go off once.

| Field | What it does | Ships at |
|---|---|---|
| `Cable Mesh` | The mesh repeated along the wire | `SM_Tripwire_Cable` |
| `Trigger Thickness` | How thick the invisible trigger boxes are, in centimetres | `4` |
| `Trigger Step` | Spacing between trigger boxes along the wire | `40` |
| `Explosion Radius` | Radius pushed into the explosion it spawns | `500` |
| `Explosion Damage` | Damage pushed into the explosion it spawns | `1000` |

Unlike the zones, the tripwire ships with usable numbers. Drop one in, stretch it, and it works.

---

## Test targets: the two dummies

Two dummies ship, in `Content/TheLastTemplate/Blueprints/Environments/`. They show different things, and neither replaces the other.

`BP_TrainingDummy` is a torso on a stand, held by a physics constraint. It swings when you hit it and settles back. Its one field is `Impulse Per Damage`, at `200`: every point of damage becomes that much impulse at the point that was hit.

It only answers to point damage, meaning a bullet or a melee trace that reports where it landed. An explosion applies damage without a hit point, so the training dummy takes the damage and does not swing. That is expected, not a bug.

`BP_Dummy` is a full character with a vitals system, a hit reaction manager and a health bar above its head. It bleeds, reacts, and dies with a ragdoll. Use it to check a weapon, a hit reaction or a hazard end to end. It has no fields of its own: tune it through its components, the same ones your own characters use. See [Tune hit reactions](../damage/tune_hit_reactions.md).

Both carry the `CanBeFire` tag, so both are valid targets for a molotov or a fire pool.

---

## Firing a hazard from your own Blueprint

Every zone reads its numbers the moment it starts, so setting them after the actor exists is too late. Set them on the spawn node instead.

1. Add a `Spawn Actor from Class` node and pick the zone.
2. Feed the transform where you want it.
3. Fill the pins the node exposes for that zone.
4. Set `Instigator` to whoever caused it, so the damage is credited to them.

| Zone | Pins on the spawn node |
|---|---|
| `BP_ExplosionZone` | `Radius`, `Damage` |
| `BP_MolotovZone` | `Radius`, `Fire Damage`, `Fire Duration` |
| `BP_NailsZone` | `Radius`, `Damage`, `Duration` |

This is exactly how the throwables do it. If your hazard should be attached to a throwable rather than to a Blueprint of yours, do not spawn it by hand: add a payload to the throwable's data asset instead, which is a list of entries with a radius, an amount and a duration.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
