# Put a workbench in your level

The workbench is where the player spends **Parts** to upgrade the guns they own. You drag one actor into the level, tick the upgrades it sells, and the rest is already wired: the view blends to the bench, the selected gun lands on the table, and every upgrade the player buys is applied to that weapon class from then on.

The actor is `Content/TheLastTemplate/Blueprints/Environments/Interactable/Workbench/BP_Interactable_Workbench`.

It is a child of `BP_InteractableOverlapBase`, so it uses the same prompt and the same two ranges as every other interactable. See [How interaction works](../inventory/how_interaction_works.md) if you have not met that base yet. The guns it works on are covered in [How guns work in this template](how_guns_work.md).

---

## Place one

1. Drag `BP_Interactable_Workbench` into your level.
2. Put it against a wall, at the height a real bench would be. The mesh already sits on the floor at the actor's origin.
3. Play, walk up to it, and press the interact key.

That is enough to have a working bench. Everything below is dressing.

---

## Choose the upgrades it offers

`Available Upgrades` is an array of upgrade Data Assets, on the placed actor. Only the upgrades in this array can ever be bought at this bench.

All five that ship are in the array, from `…/DataAssets/Weapons/Upgrades/Childs/`:

- `DA_Upgrade_Pistol_FireRate`
- `DA_Upgrade_Pistol_Recoil`
- `DA_Upgrade_Pistol_Capacity`, two levels, `+4` then `+6` rounds
- `DA_Upgrade_Silencer`, one level, `-65%` noise range and a silencer mesh on the barrel
- `DA_Upgrade_Shotgun_Choke`

The bench does not filter by weapon. The screen does that for you: each upgrade lists its own `Compatible Weapons`, and a tile that does not match the gun the player selected refuses the purchase. So you can leave the full list on every bench, or cut it down if you want one bench in your level that only sells silencers.

---

## Frame the shot

The workbench carries **its own camera**. This is the one place in the template where a camera other than the player's takes over, and it is meant to be moved.

1. Select the placed workbench.
2. In the components list, pick `Workbench Camera`.
3. Move and rotate it in the viewport until the framing is what you want. What you see through it in the editor is what the player will see.

The gun is placed the same way, with `Preview Pivot`:

1. Pick `Preview Pivot` in the components list.
2. Move it to the spot on the table where the gun should lie, and rotate it so the barrel points where you want.

Leave `Preview Weapon` alone. It is an empty skeletal mesh component that the bench fills with whichever gun the player selected on the screen.

| Field | What it does | Ships as |
|---|---|---|
| `Transition Time` | Seconds the view takes to blend into the bench, and back out again | `0.6` |
| `Preview Drop Height` | Height of the small drop the gun makes onto the table when the preview changes, in centimetres | `9` |

While the screen is open the player character is hidden and their movement and look input are locked, so you do not need to hide anything yourself or build a blocking volume.

---

## The prompt

The prompt comes from the interactable base, and the workbench overrides three of its fields.

| Field | What it does | Ships as |
|---|---|---|
| `Interact Text` | The word on the prompt | `Workbench` |
| `Interact Zone Range` | How close the player must be for the key to work | `180` |
| `Show Interact Zone Range` | How close the player must be for the prompt to appear at all | `300` |
| `Use Icon` / `Interact Icon` | Show a picture instead of the text. They are alternatives, not both | `false` / empty |

`180` is much larger than the `60` most interactables use, because you want the bench to be usable from a step back rather than nose against the table.

To move the prompt up or down, select the `Interact Widget` component and move it. It is a normal widget component on the actor.

## Where Parts come from

Parts are not a special currency. They are an ordinary inventory item, `Content/TheLastTemplate/Blueprints/DataAssets/Inventory/Childs/DA_Item_Parts`, in the `CraftingMaterial` category with `Max Carry` at `999`.

The link between the item and the bench is one field: `Parts Item` on `BP_WeaponUpgradeComponent`, at `Content/TheLastTemplate/Blueprints/ActorComponents/BP_WeaponUpgradeComponent`. It ships pointing at `DA_Item_Parts`. Point it at another item and that item becomes the currency, everywhere, with no other change.

To put Parts in your level, drag in `BP_Interactable_Pickup_Item_Parts`, from `…/Interactable/Pickup/Items/Childs/`. It gives `8` per pickup. Change `Amount` on the placed actor for a bigger or smaller pile. Every other pickup works the same way, see [Place other pickups](../inventory/place_other_pickups.md).

---

## Buying an upgrade

The player picks a gun in the top row, then picks a tile. The tile is a **hold**, not a click: it fills over the level's `Craft Duration` and cancels if the button is released early. Capacity I holds for `2` seconds, the silencer for `4`, so the cost of a big upgrade is felt as time as well as Parts.

A tile refuses the purchase for one of four reasons, and says so:

| Reason | When |
|---|---|
| `Not enough Parts` | `Parts Cost` on the next level is more than the player carries |
| `Not compatible with this weapon` | The selected gun is not in the upgrade's `Compatible Weapons` |
| `Fully upgraded` | Every level of that upgrade is already bought |
| `No inventory on this owner` | The actor holding the gun has no inventory manager to pay from |

Three sounds are involved. `Nav Sound` and `Select Sound` are fields on `BP_WorkbenchWidget`, in `Content/TheLastTemplate/Blueprints/Widgets/Workbench/`, and they use the shared UI sounds from `Content/TheLastTemplate/Audios/Widgets/`. The third is `Craft Sound` on the upgrade Data Asset itself, played when the purchase lands. It is empty on all five upgrades that ship, so fill it in if you want each upgrade to sound different.

---

## What a purchase changes

The purchase is stored against the **weapon class**, not against the gun actor the player is holding. So it survives the gun being holstered, dropped and picked up again.

Two things happen the moment the hold completes:

- the stat changes on that level are applied to the gun, so the next shot already fires with the new numbers
- the attachment meshes are rebuilt on both the gun in the player's hands and the gun on their back

Which of the two shows the attachment is up to the upgrade. Each attachment carries `Visible when Equipped` and `Visible when Holstered`, and the silencer that ships has both ticked, so it is on the pistol whether it is held or slung.

If you want a character to start with upgrades already bought, that is `Initial Loadouts` on `BP_WeaponUpgradeComponent`. It is the same field on the player and on an NPC, see [Give an AI a gun and upgrades](give_an_ai_a_gun.md).

---

## Use a different bench mesh

1. Select the placed workbench.
2. Pick the `Workbench Mesh` component.
3. Set its **Static Mesh** to yours. The one that ships is `Content/TheLastTemplate/Meshes/Environments/Props/SM_Workbench`.
4. Move `Preview Pivot` and `Workbench Camera` to match the new table height.

Nothing else reads the mesh, so any prop works. A crate, a car bonnet, a kitchen counter.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
