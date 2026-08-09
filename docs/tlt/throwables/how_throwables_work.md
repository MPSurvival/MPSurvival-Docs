# How throwables work

A throwable is something the player holds, aims and throws: a brick, a bottle, a Molotov. Six of them ship with the template, and each one is described by a single Data Asset. Flight, sound, camera, carry limit and what happens on impact all come from that one file.

- The data assets: `Content/TheLastTemplate/Blueprints/DataAssets/Throwables/Childs/`
- Everything else: `Content/TheLastTemplate/Blueprints/Throwables/`

The shipped data assets are spelled `DA_Throwble_`, without the second **a**. Search for `Throwble` or you will not find them.

---

## The pieces

| Asset | What it is | One per throwable? |
|---|---|---|
| `BP_ThrowableDataAsset` | The description of the throwable. You make a child of it. | Yes |
| `BP_CosmeticThrowable` | The prop held in the hand while you aim. You make a child and give it a mesh. | Yes |
| `BP_Interactable_Pickup_Throwable` | The pickup you drop in the level. You make a child and point it at the data asset. | Yes |
| `BP_ThrowableBase` | The actor that flies, bounces and lands. Shared by all six. | No |
| `BP_ThrowablePayloadBase` | What happens on impact. Five children ship. | No, you pick from a list |
| `BP_PlayerThrowableManager` | The component on `BP_PlayerCharacter` that holds what the player carries. | No |
| `BP_TrajectoryVisualizer` | The aim arc. Its look comes from `MI_ThrowArc`. | No |

---

## From pickup to impact

1. The player picks up a `BP_Interactable_Pickup_Throwable_*` actor, or uses an inventory item that grants one.
2. `BP_PlayerThrowableManager` stores it under its slot, with a count.
3. Equipping it spawns the `Cosmetic Class` actor in the player's hand.
4. Aiming shows the arc, if `Show Trajectory` is on, and moves the camera using the aiming fields on the data asset.
5. Releasing plays `Throw Montage` and launches a `BP_ThrowableBase` with the flight settings from the data asset.
6. It lands, and every entry in `Impact Payloads` runs at the impact point.

Steps 4 and 5 are what you tune when a throw feels wrong. See [Change how a throw feels](change_how_a_throw_feels.md).

---

## Primary and secondary

A throwable has a `Type` field with three values: `None`, `Primary` and `Secondary`. The player carries one kind of throwable per type at a time: the manager keeps one slot for the primary and one for the secondary, each with its own count.

The radial menu has one slot for the primary and one for the secondary. Bomb, Nail Bomb and Molotov are primary, the ones that hurt. Bottle, Brick and Can are secondary, the ones that make noise.

The carry limit is `Max Carry` on the data asset, plus the `Carry Bonus` on `BP_PlayerInventoryManager`. That bonus is raised by the Carry Capacity skill, `DA_Skill_CarryCapacity`, so a throwable set to 2 can end up letting the player hold more without you touching the data asset.

---

## What happens when it lands

`Impact Payloads` is an array. Each row is one thing that happens, and a throwable can stack several. Every row has the same four fields.

| Field | What it does |
|---|---|
| `Payload Class` | Which of the five payloads to run |
| `Radius` | A distance in centimetres, read differently by each payload |
| `Amount` | A quantity, read differently by each payload |
| `Duration` | Seconds, only used by the payloads that leave something behind |

That is the whole extension point. A payload class has no settings of its own, everything it needs comes from the row.

| Payload | What it does | `Radius` | `Amount` | `Duration` |
|---|---|---|---|---|
| `BP_ThrowablePayload_Noise` | Reports a noise the AI can hear. Nothing is spawned. | How far the noise carries | Loudness | not used |
| `BP_ThrowablePayload_HitDamage` | Damages every actor in a sphere around the impact | The sphere | Damage per actor | not used |
| `BP_ThrowablePayload_Explosion` | Spawns a `BP_ExplosionZone` | Blast radius | Damage | not used |
| `BP_ThrowablePayload_MolotovSpawn` | Spawns a `BP_MolotovZone`, a fire pool that sticks to the floor | Pool size | Fire damage | How long it burns |
| `BP_ThrowablePayload_NailsSpawn` | Spawns a `BP_NailsZone`, a bed of nails | Bed size | Damage | How long it stays |

To write your own, make a child of `BP_ThrowablePayloadBase` and override `Execute Payload`. It receives the row, so your payload reads the same `Radius`, `Amount` and `Duration` the shipped ones do.

!!! warning
    The three zone actors ship with their `Radius` and damage fields at `0`, because the payload fills them in when it spawns them. Drag one into a level by hand and it does nothing until you set those fields yourself. See [Place hazards and traps in your level](place_hazards_and_traps.md).

---

## The six that ship

| Data Asset | Slot | `Max Carry` | On impact |
|---|---|---|---|
| `DA_Throwble_Bomb` | Primary | 1 | Noise over 2500, then a 500 blast for 1000 damage |
| `DA_Throwble_NailBomb` | Primary | 1 | Noise over 2500, 20 damage on the spot, then a 120 nail bed for 10 seconds |
| `DA_Throwble_Molotov` | Primary | 2 | Noise over 5000, 20 damage on the spot, then a 200 fire pool for 10 seconds |
| `DA_Throwble_Brick` | Secondary | 2 | Noise over 2500, 25 damage on the spot |
| `DA_Throwble_Bottle` | Secondary | 2 | Noise over 2500, 10 damage on the spot |
| `DA_Throwble_Can` | Secondary | 2 | Noise over 2500, nothing else |

The Brick and the Can have `Destroy On Impact` off, so they stay in the world and bounce. The other four have it on.

The Molotov is the only one whose noise carries past 2500. Its noise radius is 5000, so it draws enemies from twice as far as the others.

---

## Getting one into the player's hands

Two routes, and they both end at the same data asset.

- **A pickup in the level.** Drop a `BP_Interactable_Pickup_Throwable_*` child into the map. Its `Throwable Data` field names the data asset. The six children are already set up, one per throwable.
- **An inventory item.** An item can grant a throwable instead of being used. `DA_Item_Molotov` does this: its `Effect Classes` holds a child of `BP_ItemEffect_GiveThrowable`, and `Auto Use` is on so the crafted Molotov goes straight into the throwing slot.

The second route is how crafting feeds the throwables. The player crafts a Molotov item, and the item hands the throwable over on pickup.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
