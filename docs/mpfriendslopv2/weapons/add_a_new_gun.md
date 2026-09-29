# Add a new gun

A gun is three assets: an inventory item Data Asset, a held actor that carries the gun's skeletal mesh, and a weapon Data Asset that holds every number. Copy the sawed-off shotgun for all three. Then give the gun its animation Blueprint, and its fire and reload montages. You get a gun that can be picked up, fired and reloaded, without writing any logic.

If you have not read [How weapons work](how_weapons_work.md), start there.

- Weapon Data Assets: `Content/MPFriendslop/Blueprints/DataAssets/Weapon/Childs/`
- Held actors: `Content/MPFriendslop/Blueprints/Environments/Items/Wieldable/Childs/`
- Item Data Assets: `Content/MPFriendslop/Blueprints/DataAssets/Item/Childs/`

---

## Before you start

You need two meshes of your gun:

- a **skeletal mesh**, held in the hand, with two sockets: `Muzzle`, where the shot and the flash leave, and `LeftHandGrip`, where the left hand holds it. `SKM_Tool_Shotgun_SawedOff` has both.
- a **static mesh**, used by the pickup on the ground. The shotgun's is `SM_Tool_Shotgun_SawedOff`.

The shotgun's sounds, effects and camera shake can be borrowed while you get the gun working.

---

## Step 1, the weapon Data Asset

1. In `Blueprints/DataAssets/Weapon/Childs/`, duplicate `DA_Weapon_Shotgun_SawedOff`.
2. Name it after your gun, for example `DA_Weapon_Rifle`.
3. Open it. Every field already holds a working value, so you replace rather than fill in.

**Fire**

| Field | What it does | Shotgun |
|---|---|---|
| `Fire Mode` | `Single`, `Burst` or `Auto` | `Single` |
| `Rounds Per Minute` | Fastest rate of fire. The server refuses shots that come faster | `200` |
| `Burst Count` | Shots per trigger pull in `Burst` | `3` |
| `Pellets Per Shot` | Traces per shot. `1` for a rifle or a pistol | `8` |
| `Spread Degrees` | Half angle of the cone each pellet lands in | `5` |
| `Range` | Trace length, in cm | `3000` |
| `Damage Per Pellet` | Damage of each pellet that hits | `9` |
| `Ammo Per Shot` | Charge taken from the slot per shot | `1` |

`Burst` and `Auto` are built in, but no shipped gun uses them, so the shotgun is your only example.

**Fire animations**

| Field | What it does | Shotgun |
|---|---|---|
| `Fire Montage FPS` | Plays on the gun mesh in first person, for the shooter, on every shot | `AM_Shotgun_Fire` |
| `Fire Montage TPS` | Plays on the character body, in the `UpperBody` slot, for every player | `AM_Shotgun_Fire_TPS` |
| `Fire Montage World` | Plays on the gun mesh the other players see | `AM_Shotgun_Fire_World` |

Leave any of the three empty and that view simply has no fire animation. The shooter plays the FPS montage at once, with the flash and the recoil. The other players play theirs when the server confirms the shot.

**Reload**

| Field | What it does | Shotgun |
|---|---|---|
| `Reload Seconds` | Time before the slot is refilled | `3.2` |
| `Reload Montage FPS` | Plays on the gun mesh in first person, for the shooter | `AM_Shotgun_Reload` |
| `Reload Montage TPS` | Plays on the character body, in the `UpperBody` slot | `AM_Shotgun_Reload_TPS` |
| `Reload Montage World` | Plays on the gun mesh the other players see | `AM_Shotgun_Reload_World` |

**Recoil, effects and audio**

| Field | What it does | Shotgun |
|---|---|---|
| `Recoil Pitch` | How far the view kicks up per shot | `4` |
| `Recoil Yaw` | Random sideways kick, left or right | `1.2` |
| `Recoil Recovery Speed` | How fast the view comes back down | `6` |
| `Camera Shake` | Shake played on the shooter's camera | `CS_WeaponFire` |
| `Muzzle Socket Name` | Socket the flash spawns on | `Muzzle` |
| `Muzzle FX` | Niagara system at the muzzle | `NS_Shotgun_Fire` |
| `Impact FX` | Niagara system where a shot lands | `NS_Weapon_Impact` |
| `Fire Sound` | The shot | `CUE_ShotgunFire` |
| `Dry Fire Sound` | The click when the gun is empty, heard by the shooter only | `CUE_DryFire` |
| `Impact Sound` | Played where a shot lands | `CUE_Impact_Metal` |

Two things are set outside the Data Asset. The strength of the shake is on `CS_WeaponFire` in `Blueprints/Effects/`. Duplicate it if your gun should kick differently. How far away the shot is heard comes from the attenuation `ATT_Weapon_Fire` in `Audios/Attenuation/`.

---

## Step 2, the gun's animation Blueprint

The gun mesh plays its own reload and fire montages, so it needs an animation Blueprint on its skeleton.

1. Open `Blueprints/Environments/Items/Wieldable/Animations/ABP_Shotgun`. Its whole graph is a `Slot 'DefaultSlot'` node into the output pose.
2. Create an animation Blueprint for your gun's skeleton and build the same graph.
3. Make your FPS and World montages, reload and fire, on your gun's skeleton, in `DefaultSlot`.

The TPS montages play on the character, so they are made on the character's skeleton, in the `UpperBody` slot.

---

## Step 3, the held actor

1. In `Wieldable/Childs/`, duplicate `BP_WieldedShotgun` and name it, for example `BP_WieldedRifle`.
2. Select `ViewSkeletalMesh` and set its **Skeletal Mesh Asset** to your gun and its **Anim Class** to your animation Blueprint.
3. Do the same on `HandSkeletalMesh`.
4. Select `BP_WeaponComponent` and set `Weapon Data` to your Data Asset from step 1.
5. Select `ViewMesh` and move it until the gun sits right in first person: `ViewSkeletalMesh` follows it. The shotgun's is `(30, 18, -19)` with a yaw of `90`, read from the player's eye (X forward, Y right, Z up).

Duplicate the shotgun rather than making a child of `BP_WieldedItem`. `BP_WieldedItem` shows the item's static mesh in the hand. `BP_WieldedShotgun` is already set up to hide it and show the skinned meshes instead.

`ViewSkeletalMesh` is the first person copy only the shooter sees. `HandSkeletalMesh` is the copy every other player sees. The flash is played on whichever one the viewer can see, which is why both need the `Muzzle` socket.

---

## Step 4, the item Data Asset

1. In `Blueprints/DataAssets/Item/Childs/`, duplicate `DA_Item_Shotgun_SawedOff`.
2. Set `Display Name`, `Icon` and `World Mesh` (your static mesh).
3. Set `Wield Actor Class` to your held actor from step 3.
4. Set `Max Charge` to the size of the magazine. The shotgun's is `2`: two shells.
5. Set the hold pose and the hand IK, as in [Make a held item, its hold pose and hand IK](../items/make_a_held_item.md). Keep `Left Hand IK Socket` on `LeftHandGrip` if your mesh uses that name.

Ammo is the slot's charge. Each shot takes `Ammo Per Shot` from it, and a reload fills it back to `Max Charge`. There is no ammo item and no reserve to manage.

`Use Type` stays on `Use`: firing is an item use.

---

## Step 5, put it in the game

- **In the level:** drag `BP_ItemPickup` in, set `Item Data` to your item and `Charge` to the rounds it starts with (`2` for a loaded shotgun).
- **In the shop:** see [Add an item to the shop](../items/the_shop.md). A gun bought in the shop arrives full.

Fire is the item use key, `Right Mouse` (`IA_UseItem`). Reload is `R` (`IA_Reload`). Players rebind both in the CONTROLS tab. To change the shipped keys, see [The controls, and adding an input](../start/controls.md).

---

To turn a prop into a club instead, see [Turn any grabbable object into a melee weapon](make_a_melee_weapon.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
