# How items and the inventory work

An item is a tool that goes in a slot: the flashlight, the medkit, the spare battery, the sawed-off shotgun. Every item is one Data Asset, and one component on the player, `BP_InventoryComponent`, holds the slots.

- The item Data Assets: `Content/MPFriendslop/Blueprints/DataAssets/Item/Childs/`
- The pickup and the held actors: `Blueprints/Environments/Items/`
- The components: `Blueprints/ActorComponents/`

This page is the mental model. The pages after it are the recipes.

---

## An item is not loot

The template has two families of objects, and they never mix.

| | Loot | Items |
|---|---|---|
| How you carry it | In your hands, with the grab system | In an inventory slot |
| What it is for | Extracted for its value | Used: light, heal, shoot |
| Built from | `BP_LootBase`, `BPI_Grabbable` | `BP_ItemPickup`, `BPI_Item` |

The baseball bat and the hammer are loot you grab and swing, not items. Loot is covered in [How a run works](../loot/how_a_run_works.md).

---

## What one item is made of

An item is one `DA_Item_*`, a child of `BP_ItemDataAsset`. In the world it is always the same actor, `BP_ItemPickup`, which takes its mesh from the Data Asset. In your hand it is the item's `Wield Actor Class`, a child of `BP_WieldedItem`.

| Field | What it does |
|---|---|
| `Display Name` | The prompt text on the pickup |
| `Icon` | The slot icon in the HUD |
| `Max Stack` | How many fit in one slot |
| `Use Type` | `None`: cannot be used. `Use`: the item stays. `Consume`: one is removed from the stack |
| `Max Charge` | The size of the item's resource, battery or ammo. `0` means no resource and no charge strip on the slot |
| `Pickup Class` | The actor spawned when the item is dropped or delivered |
| `Wield Actor Class` | The actor spawned in your hand while the slot is selected |
| `World Mesh` | The mesh on the ground and in the hand |
| `Restored Vital`, `Restore Amount` | Which vital one use restores, and how much. Empty heals nothing |

The hold pose, the hand sockets and hand IK are the other fields of the same Data Asset. They are on [Make a held item, its hold pose and hand IK](make_a_held_item.md).

What ships:

| Data Asset | `Use Type` | What a use does |
|---|---|---|
| `DA_Item_Flashlight` | `Use` | Turns the light on or off. `Max Charge` `100`, drained while lit |
| `DA_Item_Medkit` | `Consume` | Restores `50` health. Stacks to `3` |
| `DA_Item_Battery` | `Consume` | "Spare Battery". Restores `40` stamina. It does not recharge the flashlight |
| `DA_Item_Shotgun_SawedOff` | `Use` | Fires. `Max Charge` `2`: the charge is the two shells |

---

## The inventory on the player

`BP_InventoryComponent` sits on `BP_FriendslopCharacter`. Its fields are in `Settings|Inventory`:

| Field | What it does | Shipped default |
|---|---|---|
| `Slot Count` | Number of slots. The HUD builds itself from it | `4` |
| `Drop Distance` | How far in front of your eyes a dropped item lands, in cm. A wall in the way pulls it back | `120` |
| `Wield Socket Name` | The hand socket for an item whose Data Asset names none | `hand_r` |

What the player does with it:

- **Pick up.** Look at a `BP_ItemPickup` and press `E`. The item stacks on a slot that holds the same item first, then takes a free slot. With no room, it stays on the ground.
- **Select.** `1` to `4`, or the mouse wheel. The wheel stops at the first and last slot. Pressing the key of the slot already selected puts the item away. While you carry loot, the wheel moves the loot instead. Players can rebind the four slot keys in the CONTROLS tab, one row per slot. The wheel is not rebindable.
- **Use.** `Right Mouse` uses the selected item.
- **Drop.** `G` drops one of the selected item. The rest of the stack stays in the slot. The charge goes with it, so a half empty flashlight is still half empty when someone picks it up.
- **Die.** `BP_DeathComponent` calls `DropAll`, and everything you held falls where you died.

The charge belongs to the slot, not to the held actor. Switching slots keeps your battery and your ammo.

---

## The dispatchers

Bind these to add sounds, a tutorial or your own HUD without opening the component.

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnInventoryChanged` | A slot changed. Read the slots on the component | every player |
| `OnSelectedSlotChanged` | Another slot was selected. Gives `NewIndex`, `-1` when nothing is held | every player |
| `OnWieldedActorChanged` | The actor in the hand changed | every player |
| `OnItemUsed` | An item was used. Gives `ItemData` | every player |
| `OnInventoryFull` | A pickup found no room. Gives `ItemData` | owning player |
| `OnLocalUsePressed`, `OnLocalUseReleased`, `OnLocalReloadPressed` | The key was pressed or released, before the server answers. For instant feedback only | owning player |

!!! warning
    `OnItemUsed` fires on every machine. A listener that changes the game, such as healing, draining a charge or spawning something, must check `Has Authority` first. Without it, the change also runs on every client, where it does not belong.

The functions you call from your own Blueprints, all on the server: `TryAddItem` to give an item, `DropSlot` to drop some of a slot in front of the player (`Amount` is how many), `DropAll` from your own death system, `ConsumeSelectedCharge` and `RefillSelectedCharge` for the selected item's charge. `GetSelectedChargeRatio` can be read on any machine.

---

## On your own character

Add `BP_InventoryComponent` to your pawn. It also needs a skeletal mesh with the hand sockets, a Camera component, `BP_InteractionComponent` to pick things up, and the `IMC_Default` mappings. For items that restore a vital, add `BP_ConsumableComponent` too, and have the pawn implement `BPI_VitalManagerInterface` so it finds the vitals. The slot bar, `WBP_Inventory`, is part of the template HUD and finds the component by itself. The full list is on [How the player character works](../player/how_the_player_works.md#put-it-on-your-own-character).

---

## Where to go next

- [Add an inventory item](add_an_item.md)
- [Make a held item, its hold pose and hand IK](make_a_held_item.md)
- [Add an item to the shop](the_shop.md)
- [How weapons work](../weapons/how_weapons_work.md)

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
