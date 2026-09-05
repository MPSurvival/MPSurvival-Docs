# Zones: where things are allowed to go

A zone is a volume with a name on it. Structures declare which zones they accept, and placement refuses anywhere else.

**Without at least one zone volume in your level, nothing can be placed anywhere.**

---

## The volume

`BP_StoreZoneVolume` is a box with one field: which `E_StoreZone` it is.

| Value | What it usually covers |
|---|---|
| `SalesFloor` | The shop floor customers walk around |
| `StorageRoom` | The back room |
| `DeliveryBay` | Where boxes land |
| `Outside` | The pavement in front of the window |
| `Office` | The back office |

Three are placed in `L_ExampleMap`: the sales floor, the office, and a strip of pavement outside the window.

Zones can overlap, and a structure can allow several. The ad panel easel allows `SalesFloor` and `Outside`, so it goes in the window or on the pavement.

---

## Sizing a volume

Match the box to the **inside faces** of the room, not to the wall centre lines. In the shipped level every interior wall face sits on a multiple of 50, the placement grid, so the full cell against each wall is usable. Walls drawn on the module lines with their thickness centred leave that cell half inside the wall, and every shelf pushed against a wall is then out by 15 cm.

Two practical notes:

- Give the volume enough **height**. A ceiling fan is placed against a ceiling and still has to be inside the zone.
- Do not stretch the sales floor over the street. Customers walk from the spawner to the door across ground you do not want anyone building on.

---

## The one trap

**A zone volume must not block the `Visibility` channel.**

Customers choose where to stand by tracing from eye height to the shelf. A volume that blocks `Visibility` blocks that trace, every shelf is struck off, and nobody in the store shops at all — they walk in and leave, with nothing logged.

If your customers stop shopping right after you add a volume, this is it.

---

## Adding a zone of your own

1. Add a value to `E_StoreZone`.
2. Place a `BP_StoreZoneVolume` and set it to the new value.
3. Add the value to `AllowedZones` on the structures that belong there.

A zone no structure allows is a volume nobody can build in, which is a valid thing to want: a corridor, a staff area, an aisle you want kept clear.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
