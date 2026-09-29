# Make any actor grabbable

By the end of this page, players can pick up your actor with the left mouse button, carry it with physics and drop it again, the same way they carry loot. You need one physics mesh, a few settings, and the `BPI_Grabbable` interface. You do not need to change the player.

If you have not read [How grabbing works, and tuning the weight](how_grabbing_works.md), start there.

- The interface: `Content/MPFriendslop/Blueprints/Interfaces/BPI_Grabbable`
- The ready-made loot: `Blueprints/Environments/Loot/BP_LootBase` and its children in `Blueprints/Environments/Loot/Childs/`

---

## The quick way: a child of the loot base

If your object is loot, do not build it from scratch. Duplicate the `BP_Loot_*` child closest to yours, or make a new child of `BP_LootBase`.

1. Set the `Static Mesh` on `Mesh`.
2. On `Mesh`, in the **Physics** section, type the weight in `Mass (kg)`. The override is already ticked on `BP_LootBase`.
3. Fill `Loot Data` on its `BP_LootValueComponent`.

`BP_LootBase` already implements `BPI_Grabbable` and has every setting from the next section. A child sets only its mesh, its mass and its loot data. The loot side is covered in [Add a loot item](../loot/add_a_loot_item.md).

---

## Your own actor

Use this when the object is not loot: a prop, a key part, a body. `BP_PlayerHead` in `Blueprints/PlayerCharacter/` is the example that ships: it is grabbable without being a child of `BP_LootBase`.

1. Make a Blueprint with parent `Actor`. Put your Static Mesh component at the root.
2. On the mesh, tick `Simulate Physics`.
3. In the **Physics** section, tick the override next to `Mass (kg)` and set the weight.
4. In **Collision**, set a `Custom` profile with `Collision Enabled` on `Query and Physics`, object type `PhysicsBody`, `Visibility` on `Block` and `Pawn` on `Ignore`.
5. Still in **Collision**, set `Can Character Step Up On` to `No` and tick `Use CCD`.
6. In the actor's **Class Defaults**, under **Replication**, tick `Replicates` and `Replicate Movement`, and set `Physics Replication Mode` to `Predictive Interpolation`.
7. In **Class Settings**, add `BPI_Grabbable` to **Implemented Interfaces**.
8. Add a variable `Current Holder` of type `Actor`, set to `Replicated`.
9. Fill in the interface functions from the table below.

That copies the collision of `BP_LootBase`. `Pawn` on `Ignore` is what lets players walk through the object instead of kicking it or standing on it.

---

## The interface functions

| Function | What to return or do | If you leave it empty |
|---|---|---|
| `CanGrab` | `true` when nobody holds the object, or when `Grabber` is already the holder. `BP_PlayerHead` also refuses once the head sits in a revive bay. | Returns `false`: nobody can ever grab it. |
| `GetGrabMode` | `Free`, for a physics carry. | Returns `Free`, which is what you want. |
| `Grab` | Set `Current Holder` to `Grabber`. | Nothing remembers who holds it. |
| `Release` | If `Current Holder` is `Grabber`, clear it. | The holder never clears. |
| `GetGrabHolder` | Return `Current Holder`. | Returns nothing. A melee weapon never finds who swings it, so it deals no damage. |
| `GetGrabUpright` | `Align Upright` `true`, an `Upright Offset` and a `Max Lean Angle`, when the object must stay upright in the hand. `BP_LootBase` returns its own three fields of the same names. | The object keeps the angle it had when you grabbed it. |
| `GetGrabMotionComponent` | Nothing. | The mesh you grabbed is used. |

`UpdateGrabAim` is only for the `Axis` mode of levers and drawers, covered in [Make your own door, switch or lever](../interaction/make_your_own_door_or_lever.md).

---

## The settings that break it

| Setting | Where | If it is wrong |
|---|---|---|
| `Simulate Physics` | the mesh | The server refuses the grab. |
| `Visibility` on `Block` | the mesh collision | The player's trace never sees the object: no open hand, no grab. |
| `Replicates` | the actor | What the server does to the object never reaches the clients. |
| `Physics Replication Mode` | the actor | Clients see the object stutter. |
| `CanGrab` | the interface | Left empty, it answers `false`. |

!!! warning
    `Physics Replication Mode` is not set for you by the grab component. `BP_LootBase` and `BP_PlayerHead` set it to `Predictive Interpolation`. Your own actor keeps the engine default until you change it, and nothing in the log tells you.

---

To turn your new grabbable into a weapon, see [Turn any grabbable object into a melee weapon](../weapons/make_a_melee_weapon.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
