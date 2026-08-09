# Place ammo, weapon, throwable and gear pickups

Everything the player can take off the floor is a child of `BP_Interactable_PickupBase`. The class you drag into the level decides **what kind** of thing the player gets, and the field you fill on that actor decides **which** one.

They all live in `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/`.

---

## The families at a glance

| Drag this | The player gets | The field you fill |
|---|---|---|
| `BP_Interactable_Pickup_Item` | a bag item | `Item Data` |
| `BP_Interactable_Pickup_Ammo` | rounds for a gun | `Weapon Needed Classes` and `Amount` |
| `BP_Interactable_Pickup_WeaponBase` | a gun | `Weapon Class` |
| `BP_Interactable_Pickup_Throwable` | a throwable | `Throwable Data` |
| `BP_Interactable_Pickup_InspectableBase` | an object to inspect | `Inspect Data` |
| `BP_Interactable_Backpack` | the backpack | nothing |
| `BP_Interactable_Light` | the flashlight | nothing |

Most of these bases have a `Childs` folder beside them holding ready-made variants. A child sets the Data Asset, the mesh and the prompt text and adds nothing else, so placing one is a single drag with no fields to fill. Make a new variant by duplicating a child and swapping those values. The backpack and the light have no children, because there is only one of each.

Bag items are covered on their own page, [Add a new item to the inventory](add_an_item.md). Everything else is below.

---

## Ammo boxes

Ammo is the one family that is not driven by a Data Asset. An ammo box carries a **list of weapon classes it feeds** and a count.

| Field | What it does |
|---|---|
| `Weapon Needed Classes` | the guns this box gives rounds to. The player must own one of them |
| `Amount` | how many rounds the box holds. `0` on the base class |

Two are ready to place:

| Child | `Weapon Needed Classes` | `Amount` |
|---|---|---|
| `BP_Interactable_Pickup_Ammo_Pistol` | `BP_Weapon_Pistol_01`, `BP_Weapon_Pistol_02` | `8` |
| `BP_Interactable_Pickup_Ammo_Shotgun` | `BP_Weapon_Shotgun_01`, `BP_Weapon_Shotgun_02` | `4` |

They carry `SM_AmmoBox_Pistol` and `SM_AmmoBox_Shotgun`, with the prompt text "Pistol ammo box" and "Shotgun ammo box".

The ammo pickup class is also the **ammo type**, and two other places point at it:

- the gun's Data Asset has a `Munition Class` field. `DA_Pistol_01` has it set to `BP_Interactable_Pickup_Ammo_Pistol`.
- `BP_PlayerWeaponManager` has a `Max Ammo` map keyed by that same class. It ships with `16` for pistol ammo and `8` for shotgun ammo. That is where the reserve cap of an ammo type is set.

So a new calibre is three edits: a new child of `BP_Interactable_Pickup_Ammo`, the `Munition Class` of the gun that uses it, and a row in `Max Ammo`. See [Add a new gun](../weapons/add_a_new_gun.md) and [Ammo and holstered guns](../weapons/ammo_and_holstered_guns.md).

---

## Weapon pickups

`BP_Interactable_Pickup_WeaponBase` adds exactly one field, `Weapon Class`, and it takes the weapon Blueprint itself.

| Child | `Weapon Class` | `Static Mesh` |
|---|---|---|
| `BP_Interactable_Pickup_Pistol_01` | `BP_Weapon_Pistol_01` | `SM_Pistol` |
| `BP_Interactable_Pickup_Pistol_02` | `BP_Weapon_Pistol_02` | `SM_Pistol` |
| `BP_Interactable_Pickup_Shotgun_01` | `BP_Weapon_Shotgun_01` | `SM_Shotgun_Collision` |
| `BP_Interactable_Pickup_Shotgun_02` | `BP_Weapon_Shotgun_02` | `SM_Shotgun_02_Collision` |

The two pistols prompt with "Pistol" and the two shotguns with "Shotgun".

`Static Mesh` and `Weapon Class` are separate on purpose. The mesh is only what lies on the ground, the class is what the player ends up holding, and you can point the pickup at any mesh you like.

Placing a second copy of a gun the player already owns is safe: taking it adds its ammo instead of a duplicate weapon. That makes a weapon pickup a fine way to top up a player who backtracked.

---

## Throwable pickups

One field, `Throwable Data`, taking a `BP_ThrowableDataAsset`.

| Child | `Throwable Data` | `Static Mesh` |
|---|---|---|
| `BP_Interactable_Pickup_Throwable_Bomb` | `DA_Throwble_Bomb` | `SM_Bomb` |
| `BP_Interactable_Pickup_Throwable_Bottle` | `DA_Throwble_Bottle` | `SM_Bottle` |
| `BP_Interactable_Pickup_Throwable_Brick` | `DA_Throwble_Brick` | `SM_Brick` |
| `BP_Interactable_Pickup_Throwable_Can` | `DA_Throwble_Can` | `SM_Can` |
| `BP_Interactable_Pickup_Throwable_Molotov` | `DA_Throwble_Molotov` | `SM_Molotov` |
| `BP_Interactable_Pickup_Throwable_NailBomb` | `DA_Throwble_NailBomb` | `SM_NailBomb` |

The Data Assets really are spelled `DA_Throwble_`, without the second "a". Search for that or you will not find them.

There is no `Amount` on a throwable pickup. How many of a throwable the player may hold is `Max Carry` on the Data Asset, which is `2` on `DA_Throwble_Molotov`.

All six children also tighten the prompt distances to `Interact Zone Range` `50` and `Show Interact Zone Range` `150`, against `100` and `200` on the base. A throwable lying next to an item pickup therefore offers itself a little later. Keep those values if you add a seventh.

To build a new throwable rather than just place one, read [Add your own throwable](../throwables/add_your_own_throwable.md).

---

## The backpack and the flashlight

These two add no properties at all. They are `BP_Interactable_PickupBase` with a mesh and a prompt.

| Actor | `Static Mesh` | `Interact Text` |
|---|---|---|
| `BP_Interactable_Backpack` | `SM_Backpack` | Backpack |
| `BP_Interactable_Light` | `SM_Flashlight` | Light |

The backpack is a gate. Until the player has taken it, item pickups refuse and the inventory screen will not open. Ammo boxes, weapons and throwables do **not** need it, so a level can hand out a gun before it hands out the bag. Place the backpack before the first item you expect the player to keep.

---

## Books and other objects to inspect

`BP_Interactable_Pickup_InspectableBase` takes one field, `Inspect Data`, and opens the inspect view instead of giving anything. It overrides the prompt to show an icon: `Interact Text` is `Examine`, `Use Icon` is on and `Interact Icon` is `T_Journal`.

| Child | `Inspect Data` | `Static Mesh` |
|---|---|---|
| `BP_Interactable_Pickup_Book_Field` | `DA_Inspect_Book_Field` | `SM_Book_Field` |
| `BP_Interactable_Pickup_Book_Gunsmith` | `DA_Inspect_Book_Gunsmith` | `SM_Book_Gunsmith` |
| `BP_Interactable_Pickup_Book_Medical` | `DA_Inspect_Book_Medical` | `SM_Book_Medical` |

What happens when the player presses a button inside the inspect view lives on the Data Asset, not on the pickup. That is [Add an object the player can inspect](../progression/add_an_inspectable_object.md).

---

## The fields every pickup shares

These come from `BP_Interactable_PickupBase` and from the interaction base above it, so they are on all of the above.

| Field | What it does | Shipped value |
|---|---|---|
| `Static Mesh` | the mesh shown in the level | `EditorCube` |
| `Use Physics` | the object simulates and can be knocked around | `true` |
| `Use Overlay Material` | draws the highlight around it | `true` |
| `Show Amount` | appends the count to the prompt, as `(x8)` | `false`, and `true` on item and ammo pickups |
| `Interact Text` | the prompt text | empty |
| `Error Text` | the refusal text, unless the child returns its own | `Full` |
| `Use Icon` | show `Interact Icon` in place of `Interact Text` | `false` |
| `Interact Icon` | the icon to show | empty |
| `Interact Zone Range` | distance at which taking becomes possible | `100` |
| `Show Interact Zone Range` | distance at which the prompt appears | `200` |

`Static Mesh` is a **property on the actor**, not the mesh slot of the `Pickup Mesh` component. Fill the property. The construction script pushes it into the component, and anything you set directly on the component is overwritten.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
