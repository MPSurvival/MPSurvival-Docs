# Add your own throwable

Six throwables ship with the template. This page adds a seventh: an object the player can find in the level, hold, aim, throw, and that does what you decide when it lands.

You will make three assets and fill in fields. No graph to open.

---

## Before you start

Have ready:

- a **static mesh** for the object
- a **HUD icon** texture, white on transparent, like the ones in `Content/TheLastTemplate/Textures/Widgets/Icons/`
- optionally a **sound** for the bounce and the hit

Then pick the shipped throwable closest to what you want. You duplicate that one, so start from the one whose impact is nearest yours.

| Duplicate this | Slot | What it does when it lands |
|---|---|---|
| `DA_Throwble_Can` | Secondary | Noise only, keeps bouncing |
| `DA_Throwble_Brick` | Secondary | Noise, 25 damage on contact, keeps bouncing |
| `DA_Throwble_Bottle` | Secondary | Noise, 10 damage on contact, breaks |
| `DA_Throwble_Bomb` | Primary | Noise, then an explosion |
| `DA_Throwble_Molotov` | Primary | Noise, damage, then a pool of fire |
| `DA_Throwble_NailBomb` | Primary | Noise, damage, then nails on the ground |

The Data Assets are spelled `DA_Throwble_`, without the second **a**. That is how they ship. Search for `Throwble` and you will find them, in `Content/TheLastTemplate/Blueprints/DataAssets/Throwables/Childs/`.

---

## Step 1, duplicate the data asset

The data asset is the throwable. Everything else points back at it.

1. In `Content/TheLastTemplate/Blueprints/DataAssets/Throwables/Childs/`, right click the one you picked and choose **Duplicate**.
2. Name it after your object, for example `DA_Throwble_Flare`.
3. Open it and set `Display Name`. This is the name the player reads.
4. Leave `Throwable Actor Class` on `BP_ThrowableBase`. All six shipped throwables use it. It is the actor that flies, bounces and reports the impact.
5. Leave `Throw Montage` on `AM_Character_Throw` unless you animated your own.
6. Leave `Throwable Socket Name` on `ThrowableSocket`.

You now have a throwable that behaves like the one you copied. The next two steps give it its own look.

---

## Step 2, make the cosmetic actor

The cosmetic is the visible object: the mesh the player sees.

1. In `Content/TheLastTemplate/Blueprints/Throwables/Cosmetics/Childs/`, right click `BP_CosmeticThrowable` and choose **Create Child Blueprint Class**. Name it `BP_Cosmetic_Flare`.
2. Open it, select the `Mesh` component, and set your static mesh.
3. Scale and rotate the `Mesh` component until the object sits right in the hand.
4. Compile and save.
5. Back in your data asset, set `Cosmetic Class` to `BP_Cosmetic_Flare`.

Child cosmetics carry no fields of their own, only the mesh. `BP_Cosmetic_Molotov` is the exception: it holds an extra fire effect for the lit rag.

---

## Step 3, make the world pickup

The pickup is the version lying in the level that the player walks up to and takes.

1. In `Content/TheLastTemplate/Blueprints/Environments/Interactable/Pickup/Throwables/Childs/`, right click `BP_Interactable_Pickup_Throwable` and choose **Create Child Blueprint Class**. Name it `BP_Interactable_Pickup_Throwable_Flare`.
2. Open it and set `Throwable Data` to your `DA_Throwble_Flare`.
3. Set `Static Mesh` to the same mesh you used on the cosmetic.
4. Set `Interact Text` to what the prompt should read, for example "Flare".
5. The six shipped children set `Interact Zone Range` to `50` and `Show Interact Zone Range` to `150`. Keep those unless the object is much bigger.
6. Compile and save.
7. Back in your data asset, set `Pickup Class` to `BP_Interactable_Pickup_Throwable_Flare`.

!!! warning "The link goes both ways"
    The data asset points at the pickup with `Pickup Class`, and the pickup points back at the data asset with `Throwable Data`. Set both. A pickup with an empty `Throwable Data` still places in the level and still shows its mesh, it just gives nothing when you take it, with no error.

---

## Step 4, choose what happens on impact

`Impact Payloads` is an array. Each row is one effect fired when the throwable lands, and a throwable stacks as many as you want. That is how one Molotov makes noise, hurts what it hits, and leaves fire behind.

Each row has four fields:

| Field | What it does |
|---|---|
| `Payload Class` | Which effect to fire |
| `Radius` | A distance in centimetres |
| `Amount` | The payload's main number |
| `Duration` | How long it lasts, in seconds. `0` for the payloads that resolve at once |

The payload classes have no settings of their own in the Details panel. `Radius`, `Amount` and `Duration` are the whole interface, and each class reads them its own way.

Five payloads ship, in `Content/TheLastTemplate/Blueprints/Throwables/Payloads/Childs/`:

| Payload | What it does | Shipped values |
|---|---|---|
| `BP_ThrowablePayload_Noise` | Makes a sound the AI can hear. Used by all six | `Radius` `2500`, `5000` on the Molotov. `Amount` `1.5` |
| `BP_ThrowablePayload_HitDamage` | Damages what is at the impact point | `Radius` `10`, `Amount` `10` to `25` depending on the object |
| `BP_ThrowablePayload_Explosion` | An explosion. Only the Bomb uses it | `Radius` `500`, `Amount` `1000` |
| `BP_ThrowablePayload_MolotovSpawn` | Leaves a pool of fire | `Radius` `200`, `Amount` `5`, `Duration` `10` |
| `BP_ThrowablePayload_NailsSpawn` | Leaves nails on the ground | `Radius` `120`, `Amount` `3`, `Duration` `10` |

To build your own stack: delete the rows you do not want, add rows with the **+** on `Impact Payloads`, and set the class and the three numbers on each.

The `Noise` radius is what the AI hears, so it is the field to raise if your object is meant to pull enemies away. See [Noise and distractions](../ai/noise_and_distractions.md).

The fire left by `MolotovSpawn` is its own system, and it can be set on other actors too. See [Make an actor catch fire](make_an_actor_catch_fire.md).

A payload class you invent is a child of `BP_ThrowablePayloadBase`, and that one does mean opening a graph. The five above cover noise, damage, blast, fire and a ground hazard, so try a combination of them first.

---

## Step 5, the slot, the carry limit and the icon

Back in the data asset:

| Field | What it does | Shipped |
|---|---|---|
| `Type` | Which of the two throwable slots this fills. `Primary` or `Secondary`. `None` means no slot | The three lethal ones are `Primary`, the three distractions are `Secondary`. Nothing forces that |
| `Max Carry` | How many the player can hold | `1` for the Bomb and the Nail Bomb, `2` for the other four |
| `Icon` | The HUD icon | A white mask texture such as `T_MolotovMaskWhite` |
| `Throwable Icon Scale` | Size of that icon on the HUD | `0.55` on all six. Match it and yours sits like the others |
| `Destroy On Impact` | Whether the object stays in the world after it lands | `true` for the Bomb, Bottle, Molotov and Nail Bomb. `false` for the Brick and the Can, which keep bouncing |
| `Launch Speed` | Initial speed of the throw | `1700`, `1400` on the Can, `1200` on the Brick |
| `Throw Sound` | Played on the throw | Empty on all six. Free to use |
| `Bounce Sound` | Played when it bounces | `SC_BreakGlass`, `SC_BrickHit` or `SC_MetalHit` |
| `Hit Sound` | Played when it hits hard enough | Same three sounds |

The rest of the data asset is about how the throw feels in the hand: the aim camera, the zoom, the arc, the movement speed while aiming. Those are the same on all six and they are covered in [Change how a throw feels](change_how_a_throw_feels.md).

---

## Step 6, get it into the player's hands

Two ways, and you can use both.

**Put it in the level.** Drag `BP_Interactable_Pickup_Throwable_Flare` into your map. The player walks up, presses the interact key, and gets it. That is all it takes. See [Place other pickups](../inventory/place_other_pickups.md) for the shared pickup fields.

**Make it an inventory item.** The Molotov and the Nail Bomb also exist as items, so they can be crafted and carried in the backpack. Copy that pattern:

1. Duplicate `DA_Item_Molotov` in `Content/TheLastTemplate/Blueprints/DataAssets/Inventory/Childs/`.
2. Set `Throwable Data` to your `DA_Throwble_Flare`.
3. Duplicate `BP_ItemEffect_GiveThrowable_Molotov` in `Content/TheLastTemplate/Blueprints/Inventory/Childs/GiveThrowable/Childs/`, then put your copy in `Effect Classes` in place of the Molotov one.
4. Leave `Auto Use` on `true` and `Category` on `Consumable`, the way both shipped throwable items are set.

Full detail on items is in [Add an item](../inventory/add_an_item.md).

## Before you press play

- The data asset has `Cosmetic Class` and `Pickup Class` filled.
- The pickup has `Throwable Data` pointing back at the data asset.
- Both the cosmetic and the pickup show your mesh.
- `Type` is `Primary` or `Secondary`, and `Max Carry` is at least `1`.
- `Impact Payloads` has at least one row, and no row left over from the throwable you copied.
- `Icon` is set, or the HUD slot draws empty.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
