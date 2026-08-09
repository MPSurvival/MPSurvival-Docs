# Add a new gun

A gun is two assets. A **Data Asset** holds every number and every asset reference. A **child of `BP_WeaponBase`** holds the mesh and the points where things spawn from it. When both exist the gun can be equipped, aimed, fired, reloaded, holstered, refilled and upgraded, and you never open a graph to get there.

If you have not read [How guns work in this template](how_guns_work.md), start there.

The two folders you will work in:

- Data Assets: `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Childs/`
- Weapon classes: `Content/TheLastTemplate/Blueprints/Weapons/Childs/`

---

## Before you start

You need a skeletal mesh for the gun. That is the only thing the template cannot give you.

Everything else can be borrowed from a gun that already ships while you get the weapon working, and replaced later:

- **Animations.** Two full sets ship, in `Content/TheLastTemplate/Animations/Montages/Characters/Pistol/` and `.../Shotgun/`. Point your gun at one of them and it will animate on day one.
- **Sounds.** `Content/TheLastTemplate/Audios/Weapons/`.
- **Muzzle flash.** `NS_Muzzle_Pistol` and `NS_Muzzle_Shotgun` in `Content/TheLastTemplate/Niagara/Muzzle/`.
- **Crosshair.** `MI_CrosshairPistol`, `MI_CrosshairShotgun` and `MI_CrosshairRifle` in `Content/TheLastTemplate/Materials/Widgets/Crosshair/`. The rifle one ships unused, so an automatic weapon already has a crosshair waiting for it.
- **Camera shake.** `CS_ShootCamera` and `CS_ShootCamera_Heavy` in `Content/TheLastTemplate/Blueprints/Misc/CameraShakes/`.

Pick the shipped gun closest to yours and copy from it. A pistol-like weapon copies `DA_Pistol_01`, a long gun copies `DA_Shotgun_01`.

---

## Step 1, make the Data Asset

1. In `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Childs/`, right click `DA_Pistol_01` (or `DA_Shotgun_01`) and choose **Duplicate**.
2. Name it after your gun, for example `DA_Rifle_01`.
3. Open it. Every field is already filled with a working value, so from here you are replacing, not filling in blanks.

Set the identity fields first:

| Field | What it does |
|---|---|
| `Weapon Display Name` | The gun's name in game. `DA_Pistol_01` ships `Semi-Auto Pistol`. |
| `Weapon Description` | One line of text describing the gun. |
| `Weapon Icon` | The silhouette drawn in the weapon wheel and on the ammo counter. Shipped icons live in `Content/TheLastTemplate/Textures/Widgets/Icons/`, for example `T_PistolMaskWhite`. |
| `Weapon Icon Scale` | Fits your icon in its slot. `0.65, 0.65` for the pistol, `0.99, 0.55` for the wider shotgun. |
| `Clip Size` | Rounds in a full magazine. 16 on the pistol, 8 on the shotgun. |
| `Radial Slot Row` | Which row of the weapon wheel the gun sits on. The shotgun is on row 1, the pistol on row 2. |
| `Weapon Type Socket` | The slot the player carries the gun in, `Primary` or `Secondary`. The shotgun is `Primary`, the pistol is `Secondary`. |
| `Weapon Overlay` | The upper body pose the character holds. |

Everything about rate of fire, spread, recoil, damage and aiming is in this same Data Asset and is covered in [Change how a gun feels to shoot](change_how_a_gun_feels.md). The fields that only matter when an AI is holding the gun are covered in [Give an AI a gun and upgrades](give_an_ai_a_gun.md).

!!! note
    `Weapon Overlay` is a fixed list: `Base`, `Shotgun`, `Pistol`, `Throw`, `Combat`, `Zombie`. Only `Pistol` and `Shotgun` are gun poses. Pick whichever of the two your weapon is closest to. A third gun pose means new animation work in the character's animation Blueprint, which is outside what a Details panel can do.

---

## Step 2, fill the animation fields

Six montage fields. Three play on the character, three play on the gun itself.

| Field | Plays on | Shipped in `DA_Pistol_01` |
|---|---|---|
| `Character Fire Animation` | Character | `AM_Character_Pistol_Fire` |
| `Character Empty Animation` | Character | `AM_Character_Pistol_EmptyFire` |
| `Character Reload Animation` | Character | `AM_Character_Pistol_Reload` |
| `Character Equip Animation` | Character | empty |
| `Weapon Fire Animation` | The gun | `AM_Pistol_Fire` |
| `Weapon Empty Animation` | The gun | empty |
| `Weapon Reload Animation` | The gun | empty |

The weapon montages are for a gun whose own mesh moves, a slide cycling or a pump racking. `DA_Shotgun_01` leaves all three weapon fields empty and drives its pump from the character montage instead. Leaving a field empty is allowed and simply means nothing plays.

The character montages must be built for the same skeleton as your character. If you are reusing the shipped pistol or shotgun set, they already are.

---

## Step 3, sounds and effects

| Field | What it does | Shipped in `DA_Pistol_01` |
|---|---|---|
| `Weapon Fire Sound` | The shot | `SC_Pistol_Fire` |
| `Empty Sound` | The click when the magazine is empty | `SC_DryFire` |
| `Magazine Drop Sound` | The magazine hitting the ground | empty |
| `Muzzle Flash Effect` | Niagara system spawned at the muzzle | `NS_Muzzle_Pistol` |
| `Muzzle Flash Scale` | Scales that system to your gun | `2, 2, 2` (the shotgun uses `3, 3, 3`) |
| `Crosshair Material` | The crosshair drawn while aiming | `MI_CrosshairPistol` |
| `Camera Shake` | The kick on the camera per shot | `CS_ShootCamera` |

`Ejected Shell` and `Eject Magazine Class` decide what physically leaves the gun. Both are actor classes, and both have a base you can duplicate:

- shells: `BP_EjectAmmoBase` in `Content/TheLastTemplate/Blueprints/Weapons/Ejects/`, with `BP_Eject_Ammo_Pistol_01` and `BP_Eject_Ammo_Shotgun_01` as worked examples
- magazines: `BP_EjectMagazineBase`, with `BP_Eject_Magazine_Pistol_01`

Duplicate one, swap its mesh, and point your Data Asset at it. `DA_Shotgun_01` leaves `Eject Magazine Class` empty because a pump shotgun has no magazine to drop.

---

## Step 4, sockets

Five socket names are typed into the Data Asset as text. If a name does not match a socket that exists, that attachment silently goes to the wrong place or does not happen.

| Field | What it points at | Pistol | Shotgun |
|---|---|---|---|
| `Weapon Grip Socket R` | Where the gun sits in the right hand while it is drawn | `PistolHandRSocket` | `ShotgunHandRSocket` |
| `Left Hand Magazine Socket` | Where the fresh magazine or shell is held during the reload | `InsertMunitionPistolSocket` | `InsertMunitionShotgunSocket` |
| `Ejected Magazine Socket` | Where the dropped magazine leaves from | `MagazineEject` | empty |
| `Unholster Humanoid Socket` | Where the gun hangs on the body when it is put away | `SecondarySocketHumanoid` | `PrimarySocketHumanoid` |
| `Unholster Backpack Socket` | Where it hangs on the backpack instead | empty | `PrimarySocketBackpack` |

`Use Unholster Backpack Socket` chooses between the last two. It is `false` on the pistol, which rides on the hip, and `true` on the shotgun, which rides on the backpack.

The safe move is to reuse the socket names of the shipped gun you copied. Your rifle can use `ShotgunHandRSocket` and hang in `PrimarySocketHumanoid` without you creating anything. Only add a socket when your mesh genuinely does not sit right in the existing ones.

Two more fields concern the left hand:

- `Use Custom Left IK` drives the off hand onto the gun. Both shipped guns have it on.
- `Join Target Location` is the offset it aims for, and it is only read when `Use Custom Left IK` is on. It is `300, 500, -5000` on the pistol and `100, -100, 0` on the shotgun, so treat it as a value you nudge until the hand lands, not one you calculate.

---

## Step 5, make the weapon class

1. In `Content/TheLastTemplate/Blueprints/Weapons/Childs/`, duplicate `BP_Weapon_Pistol_01` and name it after your gun, for example `BP_Weapon_Rifle_01`.
2. Open it. Select the `Weapon Skeletal Mesh` component and set its **Skeletal Mesh** to your gun.
3. In **Class Defaults**, set `Weapon Data` to the Data Asset you made in step 1.
4. Move the scene components in the viewport until they sit on your mesh.
5. Compile and save.

The components you move:

| Component | Put it |
|---|---|
| `Muzzle Fire Arrow` | At the muzzle, pointing forward along the barrel |
| `Eject Ammunition Arrow` and `Eject Ammunition Spawn` | At the ejection port, facing the way the shell should fly |
| `Eject Magazine Arrow` | At the magazine well, facing down |
| `Hand IK Weapon Left` | Where the left hand should grip |

That is the whole class. `Weapon Data` is the only value on it you set, and there is nothing else in it for you to edit.

---

## Step 6, ammo

Reserve ammo is tracked per weapon class, and the thing that identifies an ammo type is the ammo pickup class. So a gun that uses new ammo needs a new pickup class, and three references have to agree.

1. In `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/Ammo/Childs/`, duplicate `BP_Interactable_Pickup_Ammo_Pistol` and name it, for example `BP_Interactable_Pickup_Ammo_Rifle`.
2. Set its `Weapon Needed Classes` to the list of weapon classes this box feeds. The pistol box lists both `BP_Weapon_Pistol_01` and `BP_Weapon_Pistol_02`.
3. Set `Amount` to how many rounds one box gives. The pistol box gives 8, the shotgun box gives 4.
4. Set `Static Mesh` to your ammo box mesh and `Interact Text` to the prompt line, for example `Rifle ammo box`.
5. Back in your Data Asset, set `Munition Class` to this new pickup class.
6. Open `Content/TheLastTemplate/Blueprints/ActorComponents/BP_PlayerWeaponManager` and add a row to `Max Ammo`: your ammo pickup class as the key, the reserve cap as the value.

If your gun uses ammo a shipped gun already uses, skip all of this. Point `Munition Class` at the existing pickup class and add your weapon class to that pickup's `Weapon Needed Classes`.

!!! warning
    `Max Ammo` ships with exactly two rows, `BP_Interactable_Pickup_Ammo_Pistol` at 16 and `BP_Interactable_Pickup_Ammo_Shotgun` at 8. A new ammo type with no row there caps at zero: the gun fires the magazine it spawns with, and every ammo box the player walks over does nothing. There is no error message.

---

## Step 7, put it in the world

1. In `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/Weapons/Childs/`, duplicate `BP_Interactable_Pickup_Pistol_01`.
2. Set `Weapon Class` to your weapon class.
3. Set `Static Mesh` to the mesh the player sees lying on the floor. This is a static mesh, not the skeletal mesh the gun uses in hand. The pistol pickups use `SM_Pistol`, the shotgun pickups use `SM_Shotgun_Collision` and `SM_Shotgun_02_Collision`.
4. Set `Interact Text` to what the prompt says, for example `Rifle`.
5. Drag it into your level.

Drag the ammo pickup from step 6 in as well, so the player can refill.

---

## Step 8, the holstered gun

When the player puts a weapon away, the real weapon actor is not what you see on their back. A separate cosmetic actor, `BP_CosmeticWeapon`, is attached to the socket you named in step 4. It exists so the gun on the body costs nothing while it is not being used.

The mesh it wears comes from `Unequip Attached Mesh` on your Data Asset. `DA_Pistol_01` sets it to `SK_Pistol`, which is the same skeletal mesh `BP_Weapon_Pistol_01` wears in the hand, and `DA_Shotgun_01` sets it to `SK_Shotgun`. Set it to your gun's mesh and the holstered look is done.

Which socket it uses is the `Unholster Humanoid Socket` / `Unholster Backpack Socket` pair from step 4. There is more on this, and on what the player can carry at once, in [Ammo, reserves and holstered guns](ammo_and_holstered_guns.md).

---

## The fields that break a gun if you leave them empty

Everything else can be empty and the gun still works. These cannot.

| Field | Where | If it is empty |
|---|---|---|
| `Weapon Data` | your weapon class | The gun has no numbers at all. Nothing works. |
| `Skeletal Mesh` | the `Weapon Skeletal Mesh` component | An invisible gun. |
| `Clip Size` | Data Asset | Left at 0, the magazine holds nothing. |
| `Weapon Grip Socket R` | Data Asset | The gun attaches to the character mesh origin instead of the hand, so it sits at their feet. |
| `Weapon Overlay` | Data Asset | Left on `Base`, the character holds the gun with the empty-handed pose. |
| `Munition Class` | Data Asset | The gun has no ammo type, so it can never be refilled. |
| `Max Ammo` row | `BP_PlayerWeaponManager` | Ammo pickups are collected and thrown away. |
| `Weapon Type Socket` | Data Asset | Left on `None`, the gun has no carry slot. |

While you are testing, `Infinite Ammo` on `BP_PlayerWeaponManager` lets you fire without minding the reserve. Turn it back off before you ship.

---

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
