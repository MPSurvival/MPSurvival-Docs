# How guns work in this template

A gun is four things: a **Data Asset** that holds every number, a thin **actor class** that holds the mesh, a **firing component** every gun shares, and an **interface** the gun uses to talk to whoever is holding it.

Almost everything you will ever change is in the first one.

| Piece | Where | What it is for |
|---|---|---|
| The Data Asset | `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Childs/` | Every number and every asset reference for one gun |
| The weapon class | `Content/TheLastTemplate/Blueprints/Weapons/Childs/` | An actor that carries the mesh and points at the Data Asset |
| `BP_WeaponManager` | `Content/TheLastTemplate/Blueprints/ActorComponents/` | The component on the gun that fires, reloads and applies upgrades |
| `BPI_WeaponOwner` | `Content/TheLastTemplate/Blueprints/Interfaces/` | How the gun asks its holder for things, without caring who the holder is |

Two names look alike and are not the same thing. `BP_WeaponManager` sits on the gun. `BP_PlayerWeaponManager` and `BP_NPCWeaponManager` sit on the character and decide which gun is out. If you are looking for the code that pulls a weapon from the holster, it is on the character. If you are looking for the code that sends a bullet, it is on the gun.

---

## What the Data Asset holds

Every gun points at one child of `BP_WeaponDataAsset`. Open one and you see the whole weapon in a single Details panel, sorted into groups.

| Group | Some of the fields |
|---|---|
| Firing | `Clip Size`, `Fire Rate`, `Automatic Fire`, `Automatic Fire Rate`, `Amount Bullet Per Shoot`, `Base Damage`, `Bullet Max Distance` |
| Accuracy and recoil | `Ammo Spread`, `Max Ammo Spread`, `Spread Recovery Speed`, `Recoil Yaw Amount`, `Recoil Pitch Amount`, `Camera Shake`, `Force Camera Shake` |
| Aiming and camera | `Zoom Value`, `Camera FOV`, `Aim Movement Speed`, `Aim Fire Delay`, `Spring Arm Right`, `Spring Arm Right Crouching`, `Crosshair Material` |
| Animation | `Character Fire Animation`, `Character Reload Animation`, `Character Equip Animation`, `Weapon Fire Animation`, `Weapon Overlay` |
| Sound and effects | `Weapon Fire Sound`, `Empty Sound`, `Magazine Drop Sound`, `Muzzle Flash Effect`, `Muzzle Flash Scale` |
| Shells and magazines | `Ejected Shell`, `Eject Magazine Class`, `Ejection Force`, `Magazine Ejection Force`, `Ejected Magazine Socket`, `Left Hand Magazine Socket` |
| Carrying it | `Weapon Type Socket`, `Unequip Attached Mesh`, `Unholster Humanoid Socket`, `Unholster Backpack Socket`, `Use Unholster Backpack Socket` |
| UI and ammo | `Weapon Display Name`, `Weapon Description`, `Weapon Icon`, `Weapon Icon Scale`, `Radial Slot Row`, `Munition Class` |
| AI | `AI Engage Range`, `AI Preferred Range`, `AI Burst Count`, `AI Spread Multiplier` |

The numbers that decide how a gun feels in the hand are covered field by field in [Change how a gun feels to shoot](change_how_a_gun_feels.md).

---

## The weapon class is thin, on purpose

`BP_Weapon_Pistol_01` and its three siblings are children of `BP_WeaponBase`. A child sets its skeletal mesh on `WeaponSkeletalMesh`, points `Weapon Data` at its Data Asset, and places the arrow components that mark where things leave the gun: `MuzzleFireArrow`, `EjectAmmunitionArrow`, `EjectMagazineArrow`. `HandIKWeaponLeft` is the point the left hand is pulled to.

The firing, the reload loop, the ammo counter and the aiming all live in the parent. That is the reason a new gun is a mesh, a Data Asset and a child class, and not a Blueprint you have to write.

---

## Who holds a gun

The player carries `BP_PlayerWeaponManager`. An NPC carries `BP_NPCWeaponManager`. Both `BP_PlayerCharacter` and `BP_NPCCharacter` implement `BPI_WeaponOwner`, so the gun asks its holder through the interface and never has to know which of the two it is attached to.

The practical result: a weapon you build for the player works on an AI with no second version and no extra fields. The AI reads the `AI` group of the same Data Asset for its range and its burst length. See [Give an AI a gun and upgrades](give_an_ai_a_gun.md).

---

## The numbers a gun really shoots with

`BP_WeaponManager` keeps a second set of values next to `Weapon Data`, all prefixed `Eff`: `Eff Damage`, `Eff Shot Interval`, `Eff Spread`, `Eff Max Spread`, `Eff Recoil Yaw`, `Eff Recoil Pitch`, `Eff Magazine Capacity`, `Eff Bullet Range`, `Eff Fire Sound`. Those are the values the shot is built from. They are the Data Asset values with the upgrades the owner has fitted applied on top.

This is why an upgrade needs no second copy of the stat list. An upgrade level names a stat, an operation (`Add`, `Multiply` or `Override`) and a value, and the effective stat is recomputed. A silencer that swaps the fire sound and a magazine extension that adds four rounds are the same kind of asset. See [Add a weapon upgrade](add_a_weapon_upgrade.md).

!!! warning
    The `Eff` fields are outputs. Typing a value into one of them in a Details panel does nothing useful: it is overwritten the next time the effective stats are recomputed. Change the Data Asset, or add an upgrade.

---

## Where a gun sits when it is put away

The same Data Asset decides that. `Weapon Type Socket` is `Primary` or `Secondary` and picks the slot. `Unequip Attached Mesh` is what is shown while the gun is away. `Unholster Humanoid Socket` is the socket on the body, `Unholster Backpack Socket` is the socket on the backpack, and `Use Unholster Backpack Socket` chooses between them.

The two shotguns that ship use the backpack socket. The two pistols use the body socket. Reserve ammo, magazines and the holster behaviour are covered in [Ammo, reserves and holstered guns](ammo_and_holstered_guns.md).

---

## The four guns that ship

| Gun | Class | Data Asset | What it is the example of |
|---|---|---|---|
| Semi-Auto Pistol | `BP_Weapon_Pistol_01` | `DA_Pistol_01` | `Clip Size` 16, one bullet per shot, `Fire Rate` 0.2, secondary slot |
| Revolver | `BP_Weapon_Pistol_02` | `DA_Pistol_02` | The automatic path: `Automatic Fire` on, `Automatic Fire Rate` 0.05 |
| Pump Shotgun | `BP_Weapon_Shotgun_01` | `DA_Shotgun_01` | `Amount Bullet Per Shoot` 8, `Ammo Spread` 200, `Fire Rate` 0.75, primary slot |
| Short Shotgun | `BP_Weapon_Shotgun_02` | `DA_Shotgun_02` | A second body: `Ammo Spread` 100, `Bullet Max Distance` 4000, its own `SK_Shotgun_02` mesh |

Both pistols share one `Munition Class`, and both shotguns share the other. Picking up shotgun ammo feeds either shotgun.

Two of the four are variants meant to be rewritten. `DA_Pistol_02` reuses the pistol mesh of `DA_Pistol_01`, so the revolver is a pistol with different firing values, not a different model. `DA_Pistol_02` and `DA_Shotgun_02` also have no icon of their own: their `Weapon Icon` points at a texture that is not a weapon icon. Give them a mesh, an icon and a `Weapon Description` that matches their numbers, or delete them and build your own from [Add a new gun](add_a_new_gun.md).

---

## What you open, and what you can leave closed

Open these:

- the Data Asset of the gun, for anything numeric, audible or visible
- the child weapon class, only to change the mesh or move the muzzle and ejection arrows
- the upgrade Data Assets in `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Upgrades/Childs/`
- `BP_Interactable_Workbench`, to choose which upgrades a bench offers, in [Put a workbench in your level](put_a_workbench_in_your_level.md)

You do not need to open `BP_WeaponBase` or `BP_WeaponManager` to add a gun, retune one, or give one to an AI. If you find yourself in either of them, check first whether the field you want already exists in the Data Asset.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
