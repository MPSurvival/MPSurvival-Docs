# Ammo, reserves and holstered guns

Every gun in the template carries two numbers: the rounds in the magazine, and the rounds in the reserve. This page tells you where each one comes from, how ammo boxes decide who may pick them up, and what the character wears on the hip and on the back when a gun is put away.

The magazine size is `Clip Size` on the weapon's Data Asset, in `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Childs/`.

The reserve lives somewhere else: on `BP_PlayerWeaponManager`, the component on `BP_PlayerCharacter` at `Content/TheLastTemplate/Blueprints/ActorComponents/BP_PlayerWeaponManager`.

---

## The reserve is per ammo type, not per gun

The player does not have a reserve for the pistol he is holding. He has a reserve for pistol ammo. The link is `Munition Class` on the weapon Data Asset: it points at an ammo pickup class, and every gun that names the same class draws from the same pool.

| Weapon Data Asset | `Weapon Display Name` | `Clip Size` | `Munition Class` |
|---|---|---|---|
| `DA_Pistol_01` | Semi-Auto Pistol | 16 | `BP_Interactable_Pickup_Ammo_Pistol` |
| `DA_Pistol_02` | Revolver | 16 | `BP_Interactable_Pickup_Ammo_Pistol` |
| `DA_Shotgun_01` | Pump Shotgun | 8 | `BP_Interactable_Pickup_Ammo_Shotgun` |
| `DA_Shotgun_02` | Short Shotgun | 8 | `BP_Interactable_Pickup_Ammo_Shotgun` |

So the two pistols share one pile of bullets, and the two shotguns share another. Drop a third pistol into your game and point its `Munition Class` at `BP_Interactable_Pickup_Ammo_Pistol` and it feeds off the same pile with no extra work.

The cap on each pile is `Max Ammo` on `BP_PlayerWeaponManager`. It is a map, keyed by the same ammo pickup class.

| Key | Shipped cap |
|---|---|
| `BP_Interactable_Pickup_Ammo_Pistol` | 16 |
| `BP_Interactable_Pickup_Ammo_Shotgun` | 8 |

Both caps are exactly one magazine. A full player carries one clip loaded and one clip spare, and nothing more. Raise the numbers here if you want a less mean survival game.

Two more maps sit next to it, `Ammo Inventory` and `Current Ammo Weapons`. Those are filled while the game runs. Do not author values in them.

`Infinite Ammo` is on the same component and ships off. Tick it while you are testing a level and the reserve stops going down.

---

## Ammo boxes

An ammo box is `BP_Interactable_Pickup_Ammo`, in `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/Ammo/`. It is not driven by a Data Asset. It is a short list of weapon classes plus a count.

| Field | What it does |
|---|---|
| `Weapon Needed Classes` | The guns this box feeds. The player must own one of them. |
| `Amount` | How many rounds the box gives |
| `Static Mesh` | The mesh the box shows in the world |
| `Interact Text` | The line the interact prompt reads |
| `Use Physics` | Whether the box falls and settles when the level starts |

Two children ship:

| Child | `Weapon Needed Classes` | `Amount` | `Static Mesh` | `Interact Text` |
|---|---|---|---|---|
| `BP_Interactable_Pickup_Ammo_Pistol` | `BP_Weapon_Pistol_01`, `BP_Weapon_Pistol_02` | 8 | `SM_AmmoBox_Pistol` | Pistol ammo box |
| `BP_Interactable_Pickup_Ammo_Shotgun` | `BP_Weapon_Shotgun_01`, `BP_Weapon_Shotgun_02` | 4 | `SM_AmmoBox_Shotgun` | Shotgun ammo box |

To make an ammo box for a gun of your own:

1. Right click `BP_Interactable_Pickup_Ammo`, then **Create Child Blueprint Class**. Name it after the ammo, not after one gun.
2. Open it and add your weapon class to `Weapon Needed Classes`.
3. Set `Amount` to the number of rounds one box gives.
4. Set `Static Mesh` and `Interact Text`.
5. Open your weapon's Data Asset and set `Munition Class` to this new child.

Step 5 is the one people forget. The box and the gun have to name each other: the box lists the weapon class, the weapon names the box class.

!!! warning "A gun with no ammo box can never be resupplied"
    If no ammo box lists your weapon class in `Weapon Needed Classes`, the player can find your gun, empty the magazine, and then be stuck with it for the rest of the level. Nothing warns you. Check that every gun you add appears in some box's list.

## Where a gun sits when it is not in your hands

`Weapon Type Socket` on the weapon Data Asset decides which of the two holsters the gun uses. It has three values, `None`, `Primary` and `Secondary`. No shipped gun uses `None`. In practice you have a long gun slot and a sidearm slot, and each holds one gun at a time.

Three more fields on the same Data Asset name the actual sockets on the character skeleton:

| Field | What it does |
|---|---|
| `Unholster Humanoid Socket` | The socket on the body the holstered copy attaches to |
| `Unholster Backpack Socket` | The socket to use instead when the gun rides on the backpack |
| `Use Unholster Backpack Socket` | Which of the two above is used |

Shipped:

| Weapon Data Asset | `Weapon Type Socket` | `Unholster Humanoid Socket` | `Unholster Backpack Socket` | `Use Unholster Backpack Socket` |
|---|---|---|---|---|
| `DA_Pistol_01` | Secondary | `SecondarySocketHumanoid` | none | false |
| `DA_Pistol_02` | Secondary | `SecondarySocketHumanoid` | none | false |
| `DA_Shotgun_01` | Primary | `PrimarySocketHumanoid` | `PrimarySocketBackpack` | true |
| `DA_Shotgun_02` | Primary | `PrimarySocketHumanoid` | `PrimarySocketBackpack` | true |

Pistols go on the hip and stay there. Shotguns go on the back, on the backpack.

`BP_PlayerCharacter` holds one more socket name of its own, `Secondary Backpack Holster Socket`, shipped as `SecundarySocketColdreBackPack`. If you rebuild the backpack mesh and rename its sockets, that field has to follow.

---

## The holstered gun is a second actor

The gun you shoot and the gun you see on the character's back are not the same actor. The holstered one is a `BP_CosmeticWeapon`, in `Content/TheLastTemplate/Blueprints/Weapons/Cosmetics/`. It carries a skeletal mesh and nothing else that matters to you.

The mesh it wears is `Unequip Attached Mesh` on the weapon Data Asset. `DA_Pistol_01` names `SK_Pistol`, `DA_Shotgun_01` names `SK_Shotgun`, `DA_Shotgun_02` names `SK_Shotgun_02`. Fill that one field on a new gun and the holstered copy is done.

Upgrades follow the gun onto the holster. An attachment on an upgrade is an `S_WeaponAttachmentSpec`, and it carries two visibility flags, `Visible When Equipped` and `Visible When Holstered`. `DA_Upgrade_Silencer` ships with `Visible When Holstered` ticked, so a silenced pistol still shows its silencer on the hip. Untick it for a part that should only appear in the hands. See [Add a weapon upgrade](add_a_weapon_upgrade.md).

---

## Putting a gun away

`IA_Unequip` puts everything away and leaves the character empty handed. The middle mouse button and the mouse wheel open the weapon wheel and switch between the guns the player owns. Both are covered in [Change the controls](../start/change_the_controls.md).

`Radial Slot Row` on the weapon Data Asset decides which row of the wheel a gun is offered in. Shipped: `DA_Pistol_01` is on row 2, the other three are on row 1.

---

## The ammo counter on screen

The counter is a widget class named on `BP_WeaponBase`, in the field `BP_AmmoCounterWidget`. It points at `BP_AmmoCounterWidget` in `Content/TheLastTemplate/Blueprints/Widgets/Gameplay/`. Point it at a widget of your own to replace the whole thing.

The magazine number it draws is not `Clip Size` directly. `BP_WeaponManager` recomputes `Eff Magazine Capacity` from `Clip Size` plus whatever upgrades are fitted, and that is the number the counter follows. So a capacity upgrade changes the counter with no work on the widget.

---

## Starting the level with a gun

The simple way is to place a weapon pickup near the player start. Four ship, in `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/Weapons/Childs/`: `BP_Interactable_Pickup_Pistol_01`, `BP_Interactable_Pickup_Pistol_02`, `BP_Interactable_Pickup_Shotgun_01` and `BP_Interactable_Pickup_Shotgun_02`. Drop an ammo box beside it and the player is armed within a few seconds of the level opening.

If you want the gun already in hand, the list of guns the player owns is `Weapons Rows` on `BP_PlayerWeaponManager`. Select the component on `BP_PlayerCharacter` and add a row. A row is an `S_PlayerWeaponCollected`:

| Field | What it does |
|---|---|
| `Weapons` | The weapon classes in this row |
| `Weapon Type` | `Primary` or `Secondary`, the holster this row uses |
| `Row Slot Index` | Which slot of the wheel this row occupies |
| `Equipped Index` | Which entry of `Weapons` is the equipped one |
| `Last Selected Index` | Which entry the wheel returns to |

The reserve is still separate. Owning the gun does not give the player rounds for it, so put an ammo box in the level whichever route you take.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
