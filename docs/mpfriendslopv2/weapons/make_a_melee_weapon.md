# Turn any grabbable object into a melee weapon

A melee weapon is a grabbed object with a `BP_MeleeComponent` on it, a melee Data Asset, and two sockets on its mesh. There is no attack key: you grab the object with `Left Mouse` and whip the mouse. No graph to open.

If you have not read [How weapons work](how_weapons_work.md), start there. It also covers what your own actor needs to take the damage.

- Melee Data Assets: `Content/MPFriendslop/Blueprints/DataAssets/Melee/Childs/`
- The component: `Content/MPFriendslop/Blueprints/ActorComponents/BP_MeleeComponent`
- The shipped weapons: `Blueprints/Environments/Loot/Childs/BP_Loot_BaseballBat` and `BP_Loot_Hammer`

---

## What makes a strike

A grabbed loot item is pulled toward the point in front of your eyes. So when you turn your view fast, the object is dragged through the air, and that is the swing. Three checks decide whether it hits:

- **A swing.** Your view must turn by at least `Swing Turn Speed` times `Swing Turn Window` degrees within `Swing Turn Window` seconds, in any direction. With the shipped values, that is 30 degrees in a tenth of a second. The swing then stays open for `Swing Duration` seconds after the last fast turn.
- **A fast tip.** While the swing is open, the component measures the speed of the `StrikeEnd` socket, in centimetres per second, relative to you. Outside a swing, that speed counts as zero.
- **A hit.** From `Min Strike Speed`, and once `Strike Cooldown` has passed since the last hit, the zone between the two sockets is swept with spheres of `Strike Radius`. Each actor it touches whose object type is in `Strike Object Types` is hit once. The weapon never hits itself or the player who holds it.

This is why walking does nothing. Without a fast turn of the view there is no swing. And your own movement is removed from the tip speed, so running into someone adds nothing to the hit. Picking the object up does not whoosh or strike either.

The faster the tip, the harder the hit. The strength of a hit runs from `0` at `Min Strike Speed` to `1` at `Max Strike Speed`, and the damage, the push, the hit sound and the camera shake all follow it.

---

## Before you start

You need an object that can already be grabbed. The simplest is a loot item, a child of `BP_LootBase`: see [Add a loot item](../loot/add_a_loot_item.md). For an actor of your own, see [Make any actor grabbable](../grab/make_an_actor_grabbable.md) first.

---

## Step 1, the strike sockets

The strike zone is the segment between two sockets on the mesh.

1. Open your static mesh and open the **Socket Manager**.
2. Add a socket named `StrikeStart` where the hitting part begins.
3. Add a socket named `StrikeEnd` at the tip. The tip speed is measured on this one.

On `SM_Loot_BaseballBat`, `StrikeStart` is at `(0, 0, 52)` and `StrikeEnd` at `(0, 0, 78)`: the last part of the bat. On `SM_Loot_Hammer`, the two sockets sit on each side of the head.

!!! warning
    If no mesh on the actor has a socket with the name in `Strike Start Socket`, or if `Melee Data` is empty, the component switches itself off. You can still grab and carry the object, but it never whooshes and never hits. There is no error.

---

## Step 2, the melee Data Asset

1. In `Blueprints/DataAssets/Melee/Childs/`, duplicate `DA_Melee_BaseballBat` or `DA_Melee_Hammer`.
2. Name it after your weapon, for example `DA_Melee_Crowbar`.
3. Open it. Every field already holds a working value, so you replace rather than fill in.

**The swing**, in `Settings|Swing`

| Field | What it does | Bat | Hammer |
|---|---|---|---|
| `Swing Turn Speed` | How fast the view must turn to count as a swing, in degrees per second | `300` | `300` |
| `Swing Turn Window` | Time over which that turn is measured, in seconds | `0.1` | `0.1` |
| `Swing Duration` | How long the swing stays open after the last fast turn, in seconds | `0.3` | `0.3` |

**The strike**, in `Settings|Strike`

| Field | What it does | Bat | Hammer |
|---|---|---|---|
| `Strike Start Socket` | Socket where the strike zone begins | `StrikeStart` | `StrikeStart` |
| `Strike End Socket` | Socket at the tip, where the zone ends | `StrikeEnd` | `StrikeEnd` |
| `Strike Radius` | Radius of the spheres swept along the zone | `5` | `4.5` |
| `Strike Cooldown` | Shortest time between two hits, in seconds | `0.4` | `0.35` |
| `Strike Object Types` | What the weapon can hit | `Pawn`, `PhysicsBody` | `Pawn`, `PhysicsBody` |

If you named your sockets something else in Step 1, type those names here.

**The damage**, in `Settings|Damage`

| Field | What it does | Bat | Hammer |
|---|---|---|---|
| `Min Strike Speed` | Tip speed below which nothing is hit | `250` | `200` |
| `Max Strike Speed` | Tip speed that gives a full-strength hit | `900` | `700` |
| `Min Damage` | Damage at `Min Strike Speed` | `10` | `12` |
| `Max Damage` | Damage at `Max Strike Speed` and above | `35` | `30` |
| `Knockback` | Speed given to the victim by a full-strength hit, along the swing, in centimetres per second. A character is launched, a physics object is pushed | `450` | `300` |

The hammer is the easier one to use: it hits from `200` and reaches full strength at `700`. The bat needs a faster swing and hits harder at full strength.

**The sound**, in `Settings|Audio`

| Field | What it does | Bat | Hammer |
|---|---|---|---|
| `Whoosh Speed` | Tip speed at which the whoosh plays, once each time the tip goes past it | `450` | `450` |
| `Whoosh Cooldown` | Shortest time between two whooshes, in seconds | `0.4` | `0.35` |
| `Whoosh Sound` | The swing through the air, played at the tip | `CUE_Melee_Whoosh` | `CUE_Melee_Whoosh` |
| `Heavy Strike Ratio` | Strength from which the heavy sound plays instead of the light one | `0.6` | `0.6` |
| `Light Strike Sound` | A weak hit | `CUE_Melee_Hit_Light` | `CUE_Melee_Hit_Light` |
| `Heavy Strike Sound` | A strong hit | `CUE_Melee_Hit_Heavy` | `CUE_Melee_Hit_Heavy` |

**The feedback**, in `Settings|Effects`

| Field | What it does | Bat | Hammer |
|---|---|---|---|
| `Strike FX` | Niagara system spawned at the point of impact, facing out of the surface | `NS_Weapon_Impact` | `NS_Weapon_Impact` |
| `Camera Shake` | Shake on the holder's camera when they hit, from half to full scale with the strength | `CS_MeleeStrike` | `CS_MeleeStrike` |

The strength of the shake itself is on `CS_MeleeStrike` in `Blueprints/Effects/`. Duplicate it if your weapon should kick differently.

---

## Step 3, the component

1. Open your actor Blueprint.
2. In **Components**, click **Add** and pick `BP_MeleeComponent`.
3. In its Details, in `Settings|Melee`, set `Melee Data` to your Data Asset.
4. Compile and save.

`Melee Data` is the only field on the component. `BP_Loot_Hammer` is the model: a loot child with a `BP_MeleeComponent` whose `Melee Data` is `DA_Melee_Hammer`.

---

## On your own actor

If your actor is not a child of `BP_LootBase`, it must implement `BPI_Grabbable`. That includes `GetGrabHolder`: the component asks that function who holds the object, and expects a pawn. If it returns nothing, the weapon finds no holder and deals no damage. Return the holder from a `Replicated` variable, as `BP_LootBase` does with `Current Holder`.

The component uses the first mesh on the actor that has the `Strike Start Socket`. So on an actor with several meshes, put the sockets on the one that moves with the hit.

Your character needs nothing for melee beyond being able to grab. The component reads the aim and the movement of whichever pawn `GetGrabHolder` returns.

---

## Two limits worth knowing

- An inventory item in your hand cannot be swung. Only a grabbed object can be a melee weapon.
- A victim takes the damage and the push. There is no hit reaction animation.

---

To make the object easier or harder to carry and swing, see [How grabbing works, and tuning the weight](../grab/how_grabbing_works.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
