# Add an inventory item

An inventory item is one Data Asset. The pickup on the ground is always the same actor, `BP_ItemPickup`, and it reads its mesh from the Data Asset. No list to register it in. A child class is only needed to put the item in a loot table.

By the end of this page you have an item that lies in the level, goes into a quick slot with its icon, shows in the player's hand, and, if you want, heals or restores stamina when used.

If you have not read [How items and the inventory work](how_items_work.md), start there.

- The Data Assets: `Content/MPFriendslop/Blueprints/DataAssets/Item/Childs/`
- The pickup: `Content/MPFriendslop/Blueprints/Environments/Items/BP_ItemPickup`
- The icons: `Content/MPFriendslop/Textures/Widgets/Tools/`

---

## Step 1, the Data Asset

1. In `Blueprints/DataAssets/Item/Childs/`, duplicate the shipped item closest to yours: `DA_Item_Medkit` for something you use up, `DA_Item_Flashlight` for something you keep.
2. Name it `DA_Item_<Name>`.
3. Fill in the fields below. They are all in `Settings|Item`.

| Field | What it does | Shipped in DA_Item_Medkit |
|---|---|---|
| `Display Name` | The interaction prompt on the pickup | `Medkit` |
| `Icon` | The picture in the quick slot. Draw it white on a transparent background. The slot tints it | `T_Icon_Tool_Medkit` |
| `World Mesh` | The mesh on the ground. The mesh in hand is set on the held actor | `SM_Tool_Medkit` |
| `Max Stack` | How many fit in one slot | `3` |
| `Use Type` | `None`: cannot be used. `Use`: the item stays in the slot. `Consume`: one is removed from the stack on each use | `Consume` |
| `Max Charge` | The size of the item's resource. `0` means no resource and no charge bar on the slot | `0` |
| `Pickup Class` | The actor spawned when the item is dropped or delivered | `BP_ItemPickup` |
| `Wield Actor Class` | The actor shown in hand while the slot is selected | `BP_WieldedMedkit` |
| `Restored Vital` | The vital a use restores. Empty means the item restores nothing | `DA_Health_Vital` |
| `Restore Amount` | How much of that vital one use gives back | `50` |

`Max Charge` is the flashlight's battery (`100`) and the shotgun's shells (`2`). The charge belongs to the slot, so switching slots keeps it.

Keep `Pickup Class` on `BP_ItemPickup`. Every held item has its own held actor in `Blueprints/Environments/Items/Wieldable/Childs/`. For an item that is only held, duplicate `BP_WieldedMedkit`, set the **Static Mesh** of its `HandMesh` and `ViewMesh` to your mesh, and point `Wield Actor Class` at it.

The `Settings|Overlay`, `Settings|Attachment` and `Settings|Hand IK` fields decide how the item is held. For an item held like the medkit, leave the pose and hand IK fields empty. Where it sits in first person is the position of `ViewMesh` in the held actor, and a big item needs to sit further away than a small one. Both are covered in [Make a held item, its hold pose and hand IK](make_a_held_item.md#place-it-in-first-person).

---

## Step 2, a consumable

A consumable needs no graph. Set three fields:

1. `Use Type` to `Consume`.
2. `Restored Vital` to `DA_Health_Vital` or `DA_Stamina_Vital`, or to a vital of your own.
3. `Restore Amount` to how much one use gives back.

That is how the medkit (`50` health) and the Spare Battery (`40` stamina) are made. The Spare Battery restores stamina. It does not recharge the flashlight, and no shipped item refills a charge. A new vital is covered in [Health, damage and new vitals](../player/health_and_damage.md).

The use itself is done by `BP_ConsumableComponent` on the player. `BP_FriendslopCharacter` already has it. On your own pawn you need all three of these:

- `BP_ConsumableComponent`
- `BP_InventoryComponent`
- the interface `BPI_VitalManagerInterface`, with `Get Vital Component` returning your `BP_VitalsSystem`

See [How the player character works](../player/how_the_player_works.md#put-it-on-your-own-character).

---

## Step 3, put it in the level

1. Drag `BP_ItemPickup` into the level.
2. Set its fields in the Details panel.

| Field | What it does |
|---|---|
| `Item Data` | Your `DA_Item_<Name>`. The mesh appears in the editor as soon as you set it |
| `Count` | How many the player picks up. `1` by default |
| `Charge` | The charge the item starts with: `100` for a full flashlight, `2` for a loaded shotgun |
| `Interaction Type` | `Simple` by default. `Hold` makes the player hold the interact key |
| `Interaction Duration` | For `Hold`, the hold time in seconds. Ignored for `Simple`. `Spam` is covered in [Make any actor interactable](../interaction/make_an_actor_interactable.md) |

---

## Step 4, put it in the loot

A loot table picks classes, so an item goes in it as a child of `BP_ItemPickup` that already holds its Data Asset. `BP_ItemPickup_Shotgun_SawedOff` is the shipped example.

1. In `Blueprints/Environments/Items/Childs/`, right click, then **Blueprint Class**, and pick `BP_ItemPickup` as the parent.
2. Name it `BP_ItemPickup_<Name>`.
3. In its Class Defaults, set `Item Data` to your `DA_Item_<Name>`, and `Charge` to what it starts with.
4. Add it as a row of a loot table, as in [Place loot spawn points and loot tables](../loot/place_loot_spawn_points.md#make-a-loot-table).

The child has no graph. It is not sold and adds nothing to the quota.

---

## What you get for free

Once the Data Asset exists and a pickup points at it:

- The pickup shows its mesh, its outline and its prompt.
- It stacks with items of the same kind up to `Max Stack`, then takes a free slot.
- The slot shows its icon, and a charge bar when `Max Charge` is above `0`.
- The item appears in the player's hand when its slot is selected.
- Dropping it spawns a pickup with the same count and charge. It is dropped with everything else when the player dies.

None of that needs a line of Blueprint.

---

To sell your item instead of placing it, see [Add an item to the shop](the_shop.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
