# Add a weapon upgrade

By the end of this page you have an upgrade the player can buy at the workbench: a name, an icon, one or more levels, a price in Parts, and either a change to the gun's numbers or a part the player can see bolted onto the weapon.

An upgrade is one Data Asset. There is no actor to make and no graph to open.

- The upgrades: `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Upgrades/Childs/`
- The class they are made from: `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Upgrades/BP_WeaponUpgradeDataAsset`

---

## What ships

Five upgrades come with the template. They are worth opening before you write your own, because between them they cover every field on the asset.

| Data Asset | `Display Name` | Fits | Levels | Cost in Parts |
|---|---|---|---|---|
| `DA_Upgrade_Pistol_FireRate` | Fire Rate | Pistol | 2 | 40, then 75 |
| `DA_Upgrade_Pistol_Recoil` | Recoil | Pistol | 2 | 30, then 60 |
| `DA_Upgrade_Pistol_Capacity` | Capacity | Pistol | 2 | 35, then 70 |
| `DA_Upgrade_Shotgun_Choke` | Choke | Shotgun | 2 | 45, then 85 |
| `DA_Upgrade_Silencer` | Silencer | Pistol and shotgun | 1 | 90 |

---

## Make the Data Asset

1. In `Content/TheLastTemplate/Blueprints/DataAssets/Weapons/Upgrades/Childs/`, right click, then **Miscellaneous**, then **Data Asset**.
2. Pick `BP_WeaponUpgradeDataAsset` as the class.
3. Name it `DA_Upgrade_Pistol_Sight`, or whatever your upgrade is.
4. Fill the fields below.
5. Save.

| Field | What it does |
|---|---|
| `Upgrade Id` | A short name used to tell this upgrade apart from the others. `Silencer`, `Pistol_Capacity`, `Shotgun_Choke` on the shipped ones. |
| `Display Name` | The name the workbench shows. Short: `Fire Rate`, `Recoil`, `Choke`. |
| `Description` | One line under it, for example "Fit a choke tube to tighten the shot pattern." |
| `Icon` | The texture drawn on the tile. The shipped ones live in `Content/TheLastTemplate/Textures/Widgets/Icons/` and are named `T_Upgrade_*`. |
| `Compatible Weapons` | The guns this upgrade can go on. |
| `Levels` | The list of steps the player buys, one after the other. |
| `Craft Sound` | A sound played when the level is bought. Empty on all five shipped upgrades, so the workbench uses its own sounds. |

`Compatible Weapons` takes the **weapon actor class**, not the weapon Data Asset. Put `BP_Weapon_Pistol_01` in it, not `DA_Weapon_Pistol_01`. The list can hold several: the Silencer names both `BP_Weapon_Pistol_01` and `BP_Weapon_Shotgun_01`, which is how one upgrade serves two guns.

---

## A level

`Levels` is an array. One entry is one purchase. An upgrade with one entry is bought once and is done, like the Silencer. An upgrade with three entries is bought three times.

| Field | What it does |
|---|---|
| `Title` | The name of this step. `Capacity I`, `Capacity II`, and so on. |
| `Description` | What this step gives, in the player's words: `+4 rounds`, `-25% recoil`, `-65% noise range`. |
| `Icon` | A texture for this level on its own. Unset on every shipped level, which makes the tile fall back to the upgrade's `Icon`. |
| `Parts Cost` | The price, in Parts. Whole number. `30` to `90` on the shipped levels. |
| `Craft Duration` | How long the player holds the button to buy it, in seconds. `2` to `4` on the shipped levels. |
| `Modifiers` | The numbers this level changes on the gun. |
| `Attachments` | The parts this level bolts onto the gun. |
| `Fire Sound Override` | A sound that replaces the gun's normal fire sound from this level on. |

Write each level as a **step from the level before it**, not as a total from the base gun. That is how the shipped upgrades are written: `Capacity I` is `Add 4` and `Capacity II` is `Add 6`, `Recoil I` is `Multiply 0.75` and `Recoil II` is `Multiply 0.8`. If you write the totals instead, the numbers in your `Description` lines will not match what the player feels.

---

## Modifiers

A modifier is three things: which stat, what to do to it, and a number.

| Field | What it does |
|---|---|
| `Stat` | Which number on the gun to touch. Twelve to choose from, listed below. |
| `Operation` | `Add`, `Multiply` or `Override`. |
| `Value` | The number. |

`Add` adds `Value` to the stat. `Multiply` multiplies it. `Override` replaces it outright.

A level can hold several modifiers. `Recoil I` holds two, one for `RecoilYaw` and one for `RecoilPitch`, because both have to move for the kick to feel smaller.

### The twelve stats

| `Stat` | What it changes on the gun |
|---|---|
| `Damage` | Damage a bullet deals. The gun's starting value is `Base Damage`. |
| `ShotInterval` | The delay between two shots. It is a delay, so a smaller number means a faster gun. |
| `RecoilYaw` | How far the camera kicks sideways per shot. Starts from `Recoil Yaw Amount`. |
| `RecoilPitch` | How far the camera kicks upward per shot. Starts from `Recoil Pitch Amount`. |
| `Spread` | The cone the first shot goes into. Starts from `Ammo Spread`. |
| `MaxSpread` | The widest that cone can grow while the player keeps firing. Starts from `Max Ammo Spread`. |
| `SpreadRecovery` | How fast the cone shrinks back once the player stops. Starts from `Spread Recovery Speed`. |
| `MagazineCapacity` | Rounds the magazine holds. Starts from `Clip Size`. |
| `AimMovementSpeed` | Walk speed while aiming. Starts from `Aim Movement Speed`. |
| `AimFireDelay` | The wait between raising the gun and being able to fire. Starts from `Aim Fire Delay`. |
| `BulletRange` | How far a bullet reaches. Starts from `Bullet Max Distance`. |
| `NoiseRange` | How far the shot is heard by the AI. This is the one the Silencer cuts. |

Those starting values all live on the gun's Data Asset. See [How guns work in this template](how_guns_work.md) for where they sit in the panel, and [Change how a gun feels to shoot](change_how_a_gun_feels.md) for what each one does to the feel of firing.

!!! warning "Half of these stats are costs, not benefits"
    `ShotInterval`, `RecoilYaw`, `RecoilPitch`, `Spread`, `MaxSpread`, `AimFireDelay` and `NoiseRange` are all better when they are smaller. A `Multiply` of `0.8` on `ShotInterval` is the shipped Fire Rate upgrade and makes the gun fire 25% faster. A `Multiply` of `1.25` would make it slower. Get the direction backwards and the upgrade quietly makes the gun worse, with no warning anywhere.

---

## Attachments

An attachment is a static mesh stuck on the gun at a socket, and it appears the moment the level is bought.

| Field | What it does |
|---|---|
| `Attachment Id` | A short name for this part. `Silencer` on the shipped one. |
| `Mesh` | The static mesh. The one that ships is `SM_Attachment_Silencer`, in `Content/TheLastTemplate/Meshes/Weapons/StaticMeshes/Attachments/`. |
| `Socket Name` | The socket on the gun's skeletal mesh to attach to. The Silencer uses `Attachment_Silencer`. |
| `Relative Transform` | Location, rotation and scale offset from that socket. All zeros and scale `1` on the Silencer. |
| `Visible When Equipped` | Show the part while the gun is in the player's hands. |
| `Visible When Holstered` | Show the part while the gun is on the player's back or hip. |

The socket has to exist on the gun before you name it here:

1. Open the gun's skeletal mesh, for example `SK_Pistol` in `Content/TheLastTemplate/Meshes/Weapons/SkeletalMeshes/Pistol/`.
2. In the skeleton tree, right click the bone the part belongs to and add a socket.
3. Name it. `Attachment_` and then the part is the naming the template uses.
4. Drag the socket where the part should sit.
5. Put that name in `Socket Name`.

Use `Relative Transform` for the small correction, not for the whole placement. Move the socket in the mesh editor first, because a socket you place by eye once serves every upgrade that ever uses it.

The two visibility flags are separate on purpose. A silencer stays on the barrel whether the gun is drawn or stowed, so both are true on the shipped one. A part that would clip through the character's back when the gun is holstered is the case for turning `Visible When Holstered` off.

---

## Replacing the fire sound

`Fire Sound Override` on a level swaps the gun's fire sound from that level on. It is what makes the Silencer sound suppressed: the level points at `SC_SilencerShoot`, in `Content/TheLastTemplate/Audios/Weapons/`.

Leave it empty on a level and the gun keeps whatever `Weapon Fire Sound` its own Data Asset holds.

The sound and the noise are two different things. `Fire Sound Override` is what the player hears. `NoiseRange` is how far the AI hears. The Silencer changes both, and it needs both: change only the sound and enemies still come from across the map.

---

## Put it on the workbench

A Data Asset sitting in the folder does nothing. Each workbench carries its own list.

1. Select the `BP_Interactable_Workbench` in your level.
2. Find `Available Upgrades` in the Details panel.
3. Add an entry and pick your Data Asset.
4. Save the level.

Two things have to agree before a tile shows up: the upgrade must be in that bench's `Available Upgrades`, and the gun the player has out must be in the upgrade's `Compatible Weapons`. If you place several benches, each one has its own list, so an upgrade you only add to one bench is only sold at that bench.

The price is paid in Parts, which is an ordinary inventory item, `DA_Item_Parts`. The currency is set once on `BP_WeaponUpgradeComponent`, in the `Parts Item` field. Change it there and every bench and every upgrade charges in the new item instead.

See [Put a workbench in your level](put_a_workbench_in_your_level.md) for placing and framing the bench itself.

---

## Starting with an upgrade already bought

`BP_WeaponUpgradeComponent` sits on `BP_PlayerCharacter` and on `BP_NPCCharacter`. Its `Initial Loadouts` field is what the character starts with, and it is empty on the component by default, so the player begins with nothing bought.

One entry is one gun. It holds a `Weapon Class` and a list of upgrades, each one an upgrade Data Asset plus the `Current Level` already reached. Fill it on the player character to hand the player an upgraded gun from the first second. For giving one to an enemy instead, see [Give an AI a gun and upgrades](give_an_ai_a_gun.md).

---

## Two worked examples

### A three level capacity upgrade

Copy `DA_Upgrade_Pistol_Capacity`, then:

1. Set `Upgrade Id` to `Rifle_Capacity` and `Display Name` to `Capacity`.
2. Put your rifle's actor class in `Compatible Weapons`.
3. Level one: `Title` `Capacity I`, `Description` `+5 rounds`, `Parts Cost` `35`, `Craft Duration` `2`, one modifier of `MagazineCapacity` / `Add` / `5`.
4. Level two: same shape, `+7 rounds`, `Parts Cost` `70`, one modifier of `MagazineCapacity` / `Add` / `7`.
5. Level three: `+10 rounds`, `Parts Cost` `130`, one modifier of `MagazineCapacity` / `Add` / `10`.
6. Leave `Attachments` empty and `Fire Sound Override` unset on all three.
7. Add it to the bench's `Available Upgrades`.

### A one level scope

1. Model the scope, import it as a static mesh next to `SM_Attachment_Silencer`.
2. Open `SK_Pistol` and add a socket on the slide, named `Attachment_Sight`.
3. Make a new Data Asset, `Upgrade Id` `Pistol_Sight`, `Display Name` `Sight`, and put `BP_Weapon_Pistol_01` in `Compatible Weapons`.
4. One level: `Title` `Sight`, `Parts Cost` `60`, `Craft Duration` `3`.
5. In `Attachments`, one entry: your mesh, `Socket Name` `Attachment_Sight`, both visibility flags true.
6. In `Modifiers`, one entry of `Spread` / `Multiply` / `0.85`, so the scope is worth its price and not only a mesh.
7. Add it to the bench.

---

## What costs time

- Putting the weapon **Data Asset** in `Compatible Weapons` instead of the weapon **actor class**. The field takes `BP_Weapon_Pistol_01`.
- Writing your levels as totals instead of steps, then wondering why level three feels like a downgrade.
- A `Multiply` above `1` on a stat that is a cost. It is a real change, it just goes the wrong way.
- Forgetting `Available Upgrades` on the bench. The asset is correct, it is simply not for sale anywhere.
- Naming a `Socket Name` the gun's skeleton does not have. Nothing appears and nothing complains.

---

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
