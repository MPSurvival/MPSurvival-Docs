# Add a loot item

A loot item is two assets: a `DA_Loot_*` Data Asset that holds its price and how fragile it is, and a `BP_Loot_*` actor that holds its mesh and its weight. Copy the toolbox, change both, and add the actor to a loot table. It then spawns, gets damaged, sells and shows on the recap.

If you have not read [How a run works](how_a_run_works.md), start there.

- The Data Assets: `Content/MPFriendslop/Blueprints/DataAssets/Loot/Childs/`
- The actors: `Content/MPFriendslop/Blueprints/Environments/Loot/Childs/`
- The meshes: `Content/MPFriendslop/Meshes/Props/Loot/`
- The loot tables: `Content/MPFriendslop/Blueprints/DataAssets/LootTable/Childs/`

---

## Before you start

Import your mesh into `Meshes/Props/Loot/` as `SM_Loot_<Name>`. Open it and give it a simple collision (a box, a sphere or a convex from the **Collision** menu). Loot is a physics object, and a mesh with no simple collision cannot simulate physics.

---

## Step 1, the Data Asset

1. In `Blueprints/DataAssets/Loot/Childs/`, duplicate `DA_Loot_Toolbox`.
2. Name it `DA_Loot_<Name>`.
3. Fill in the fields below.

| Field | What it does | Shipped in DA_Loot_Toolbox |
|---|---|---|
| `Base Value` | The price in perfect condition | `120` |
| `Fragility` | The share of the value that one very hard hit removes, from 0 to 1. `0` means the item never loses value | `0.1` |
| `Impact Speed Threshold` | Below this impact speed, in cm/s, a hit costs nothing | `250` |
| `Reference Impact Speed` | The impact speed, in cm/s, that costs exactly `Fragility` | `800` |
| `Impact Sound` | Played where the item hits, on every hit that costs value | `CUE_Impact_Metal` |
| `Display Name` | The name on the recap | `TOOLBOX` |

The shipped items all keep `250` and `800` and only change `Base Value` and `Fragility`: `0.03` on the hammer, `0.3` on the sample jar, `1.0` on the glass vase. The `Settings|Breaking` fields are for items that shatter, see [Make a loot item breakable](make_loot_breakable.md).

`Impact Sound` is played on the surface it hits. Give it an attenuation with occlusion off, like `ATT_Impact` on the shipped cues, or the surface muffles it to nothing.

Every `DA_Loot_*` is a `BP_LootDataAsset`. To start from an empty one instead of a copy, right click in `Blueprints/DataAssets/Loot/Childs/`, then **Miscellaneous**, then **Data Asset**, and pick `BP_LootDataAsset`. Its fields then start at the class defaults, so fill in every row of the table above.

---

## Step 2, the actor

1. In `Blueprints/Environments/Loot/Childs/`, duplicate `BP_Loot_Toolbox`.
2. Name it `BP_Loot_<Name>`.
3. Select `Mesh` and set its `Static Mesh` to your `SM_Loot_<Name>`.
4. On `Mesh`, in the **Physics** section, type the weight in `Mass (kg)`. Its override is already ticked.
5. Select `BP_LootValueComponent` and set `Loot Data` to your `DA_Loot_<Name>`.

That is the whole actor. `BP_Loot_Toolbox` sets only those three things. Everything else comes from `BP_LootBase`: the physics and collision, the grab, the outline, and the noise enemies hear when a hit costs value.

The mass is what makes an item hard to carry. See [How grabbing works, and tuning the weight](../grab/how_grabbing_works.md).

---

## Step 3, a loot table

1. Open `DA_LootTable_Salvage` or `DA_LootTable_Shelf`.
2. Add an element to `Entries`.
3. Set `Loot Class` to `BP_Loot_<Name>` and give it a `Weight`.

A weight of `3` is three times as likely as a weight of `1`. `DA_LootTable_Salvage` is used by the points on the floor. `DA_LootTable_Shelf` is used by the points on furniture and shelves, and leaves out the heavy items. An item that is in no table only appears where you drop it in the level by hand. More on tables in [Place loot spawn points and loot tables](place_loot_spawn_points.md).

---

## Changing the feedback

The red flash, the `-$N` number and the timing are on `BP_LootValueComponent`. Change them on the component of your actor, or on `BP_LootBase` for every item.

| Field | What it does |
|---|---|
| `Impact Cooldown` | The minimum time between two hits that cost value. `0.25` seconds, so an item landing flat is not charged once per corner |
| `Impact Flash Material` | The overlay flashed on the item when it loses value. Ships as `M_ImpactFlash` |
| `Impact Flash Duration` | How long the flash stays. `0.45` seconds |
| `Impact Value Popup Class` | The actor spawned at the hit point to show the loss. Ships as `BP_ImpactValuePopup` |

To replace the `-$N` number with your own effect, make an actor that implements `BPI_ImpactValue`, fill its `Show Impact Value` event (it receives the dollars lost as `Value`), and set it as `Impact Value Popup Class`.

To react in your own Blueprints, bind these on `BP_LootValueComponent`:

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnConditionChanged` | The condition changed. Gives `New Condition`, from 0 to 1 | every player |
| `OnImpact` | A hit cost value. Gives the `Hit`, the `Impact Speed` and the `Condition Loss`. A hit that costs nothing does not fire it | every player |
| `OnBroken` | The item shatters, just before it is destroyed | every player |
| `OnExtracted` | The item is sold in the chute, just before it is destroyed. Gives the `Value` | every player |

`Get Current Value` and `Get Display Name` on the same component give you the price right now and the name, on any machine.

!!! warning
    `Fragility` at `0` also means silent. The impact sound, the flash, the number and `OnImpact` only happen when a hit costs value, so an item with `Fragility` `0` never makes an impact sound. There is no error.

---

## Make it glow

`BP_Loot_WardenHead` and the arena crown wear an aura: a golden halo, sparkles that rise from it, and a warm light around the item. Any loot item can wear it.

1. Open your loot Blueprint. Click **Add**, then pick **Niagara Particle System Component**. Name it `Aura`.
2. Set its `Niagara System Asset` to `NS_Loot_Aura`, found in `Content/MPFriendslop/Niagara/Loot/`.
3. Move it to the centre of your mesh.

Two parameters change the look, per item, under **User Parameters** in the Details panel of the component:

| Parameter | What it does | Default |
|---|---|---|
| `AuraTint` | The colour of the halo and the sparkles. Values above 1 brighten them | gold, `(1, 0.55, 0.06)` |
| `LightColor` | The colour and strength of the light around the item | `(60, 33, 3.6)` |

The light of a particle system goes out as soon as the item leaves the screen, even on the walls you still see. For a light that stays, set `LightColor` to black and add a **Point Light** component, as `BP_Crown` does with its `Glow` light.

---

## Giving a value to your own actor

Any actor can be sold, not only children of `BP_LootBase`.

1. Make it grabbable first, with a mesh that has `Simulate Physics` ticked and the actor set to `Replicates`: see [Make any actor grabbable](../grab/make_an_actor_grabbable.md).
2. Add a `BP_LootValueComponent` and set its `Loot Data`.
3. Keep `Component Replicates` ticked on the component. It is ticked by default.

The component only listens to meshes that simulate physics when the game starts. A mesh that starts frozen and turns physics on later never takes damage.

Selling in the chute only needs the component. Two other checks need more:

- To be counted as left behind on the recap, the actor also needs `BPI_Grabbable`.
- To be seen as dropped on the trap, it needs `BPI_Grabbable` and a collision object type from the trap's `Detection Object Types`. These are `WorldDynamic`, `Pawn` and `PhysicsBody` by default.

!!! warning
    Without `Replicates` on the actor, or with `Component Replicates` unticked, the server still damages and sells the item, but no other player ever sees its condition change, the flash or the number. There is no error.

---

To make your item shatter, see [Make a loot item breakable](make_loot_breakable.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
