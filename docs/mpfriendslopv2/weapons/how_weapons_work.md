# How weapons work

The template has two kinds of weapon. A **gun** is an inventory item with a `BP_WeaponComponent` on its held actor. A **melee weapon** is a grabbed object with a `BP_MeleeComponent` on it. Both are tuned by a Data Asset, and both deal Unreal's own damage, so anything that listens to engine damage can be hurt by either.

- The components: `Content/MPFriendslop/Blueprints/ActorComponents/`
- The gun Data Assets: `Blueprints/DataAssets/Weapon/Childs/`
- The melee Data Assets: `Blueprints/DataAssets/Melee/Childs/`

This page is the mental model. The pages after it are the recipes.

---

## What one gun is made of

A gun is three assets that point at each other:

| Asset | What it is | The shipped one |
|---|---|---|
| The item | A `DA_Item_*` like any other item. Its `Max Charge` is the clip size | `DA_Item_Shotgun_SawedOff`, `Max Charge` `2` |
| The held actor | A child of `BP_WieldedItem`, named in the item's `Wield Actor Class`. It carries the `BP_WeaponComponent` | `BP_WieldedShotgun` |
| The weapon data | A `DA_Weapon_*`, set in the component's `Weapon Data` | `DA_Weapon_Shotgun_SawedOff` |

One gun ships, the sawed-off shotgun. Items in general are on [How items and the inventory work](../items/how_items_work.md).

The Data Asset is a child of `BP_WeaponDataAsset`. Its fields sit in five categories under `Settings`:

| Category | Fields | Shotgun |
|---|---|---|
| Fire | `Fire Mode`, `Rounds Per Minute`, `Burst Count`, `Pellets Per Shot`, `Spread Degrees`, `Range`, `Damage Per Pellet`, `Ammo Per Shot`, and three fire montages: `Fire Montage FPS`, `Fire Montage TPS`, `Fire Montage World` | `Single`, `200` rpm, `8` pellets of `9` damage, `5` degrees, `3000` cm |
| Reload | `Reload Seconds` and three reload montages: `Reload Montage FPS`, `Reload Montage TPS`, `Reload Montage World` | `3.2` s |
| Recoil | `Recoil Pitch`, `Recoil Yaw`, `Recoil Recovery Speed`, `Camera Shake` | `4`, `1.2`, `6`, `CS_WeaponFire` |
| Effects | `Muzzle Socket Name`, `Muzzle FX`, `Impact FX` | `Muzzle`, `NS_Shotgun_Fire`, `NS_Weapon_Impact` |
| Audio | `Fire Sound`, `Dry Fire Sound`, `Impact Sound` | `CUE_ShotgunFire`, `CUE_DryFire`, `CUE_Impact_Metal` |

`Fire Mode` comes from `E_FireMode`: `Single`, `Burst` or `Auto`. The shotgun uses `Single`. `Burst` and `Auto` are built into the component, but no shipped gun uses them.

---

## Firing, ammo and reload

- **Fire.** `Right Mouse` uses the selected item, and for a gun a use is a shot. Each pellet is a line trace on the `Weapon` trace channel, spread inside a cone of `Spread Degrees`, and deals `Damage Per Pellet`.
- **Ammo.** The ammo is the item's charge. Each shot removes `Ammo Per Shot` from the slot. The HUD slot draws it as the charge strip. At zero, the gun plays `Dry Fire Sound` for the shooter only.
- **Reload.** `R` refills the slot to `Max Charge` after `Reload Seconds`. It does nothing when the gun is already full. The reserve is infinite: there is no ammo item to find.
- **Recoil.** Recoil kicks your aim up by `Recoil Pitch` and sideways by a random amount up to `Recoil Yaw`, then comes back at `Recoil Recovery Speed`.

The charge belongs to the slot. Drop the shotgun with one shell left, and whoever picks it up has one shell.

Each shot plays three montages: `Fire Montage FPS` on the weapon you see in your hands, `Fire Montage TPS` on your body for the other players, and `Fire Montage World` on the weapon they see. The reload does the same with `Reload Montage FPS`, `Reload Montage TPS` and `Reload Montage World`. When the gun is put away or dropped in the middle of one, the two weapon montages end with the weapon, and `BP_WeaponComponent` stops its TPS montages on the body, on every machine. An emote playing at the same time is left alone. A new gun is covered step by step on [Add a new gun](add_a_new_gun.md).

---

## What one melee weapon is made of

Any grabbed object can hit. The baseball bat and the hammer are loot, `BP_Loot_BaseballBat` and `BP_Loot_Hammer`, each with a `BP_MeleeComponent` whose `Melee Data` points at `DA_Melee_BaseballBat` or `DA_Melee_Hammer`. There is no key for a hit: you grab the object with `Left Mouse` and swing the mouse.

- **A swing.** Your view has to turn fast, at least `Swing Turn Speed` degrees per second over `Swing Turn Window` seconds. The swing then stays open for `Swing Duration` seconds. Walking or running with the object does not strike.
- **The strike zone.** The segment between two mesh sockets, `Strike Start Socket` and `Strike End Socket`, swept with spheres of `Strike Radius`. It hits only the object types in `Strike Object Types`: `Pawn` and `PhysicsBody` on the shipped weapons.
- **The damage.** The faster the tip moves, the harder it hits: from `Min Damage` at `Min Strike Speed` to `Max Damage` at `Max Strike Speed`. Below `Min Strike Speed` nothing happens. `Knockback` pushes the victim.

The fields are in `Settings|Swing`, `Settings|Strike`, `Settings|Damage`, `Settings|Audio` and `Settings|Effects` of `BP_MeleeDataAsset`. The full table and the recipe are on [Turn any grabbable object into a melee weapon](make_a_melee_weapon.md).

Two limits worth knowing: an inventory item in your hand, the shotgun included, cannot be swung as a melee weapon. And a victim takes the damage and the push, with no hit reaction animation.

---

## Make your actor take damage

A gun and a melee weapon both call Unreal's `Apply Point Damage` on the server. So your actor takes damage when two things are true:

1. **It can be hit.** For a gun, its collision must block the `Weapon` trace channel. The channel blocks by default. So this only fails when you set your actor's collision to ignore it. For a melee weapon, its object type must be in the Data Asset's `Strike Object Types`.
2. **It listens to engine damage.** Add `BP_VitalsSystem`, like the player and the Warden. Or bind `On Take Any Damage` on your actor and handle the amount yourself, on the server.

With `BP_VitalsSystem`, the actor loses health, flashes with the `Impact Flash Material`, and at 0 health the component calls `OnDeath` from `BPI_VitalManagerInterface` on its owner. The Warden implements it and disappears. That event is where a ragdoll or a drop of your own goes. The component's fields are on [Health, damage and new vitals](../player/health_and_damage.md).

!!! warning
    Nothing tells you that a trace went through your actor. If a gun never hurts it, check its collision against the `Weapon` channel first, not your damage code.

---

## Where to go next

- [Add a new gun](add_a_new_gun.md)
- [Turn any grabbable object into a melee weapon](make_a_melee_weapon.md)
- [Make a held item, its hold pose and hand IK](../items/make_a_held_item.md)

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
