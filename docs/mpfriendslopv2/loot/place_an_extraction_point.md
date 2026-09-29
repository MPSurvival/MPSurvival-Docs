# Place an extraction point

The extraction point is one actor, `BP_ExtractionTrap`: a floor hatch over a shaft. Players drop loot on it and press a button. The hatch opens and the loot falls into the chute, where it is sold. Once the quota is met, the hatch stays open and the players jump in to leave. By the end of this page you have one in your level, linked to a button.

- The trap: `Content/MPFriendslop/Blueprints/Environments/Doors/Childs/BP_ExtractionTrap`
- Its sign: `Blueprints/Widgets/World/WBP_ExtractionSign`
- The floor decals: `Materials/Environments/Decals/`

---

## How the trap decides

The trap carries four box zones. The server checks them every `Detection Interval` and sets the trap's status, an `E_ExtractionStatus`. The sign above the hatch shows it.

| Status | When | Sign |
|---|---|---|
| `Empty` | Nothing on the hatch | `Extract loot here` |
| `Blocked` | A player stands on the hatch. This wins over `Ready` | `CLEAR THE ZONE!` |
| `Ready` | Loot on the hatch and no player on it | `Press the button!` |
| `Evacuate` | The team met the quota. This wins over everything else | `JUMP IN!` |

The button only opens the hatch when the status is `Ready`. So nobody can drop a friend down the shaft to sell loot.

| Zone | What it does |
|---|---|
| `DepositZone` | Finds loot lying on the hatch |
| `SafeZone` | Finds players on the hatch. Loot inside it takes no damage. While the hatch is open for a sale, it stops players like a railing |
| `SellZone` | In the chute. Loot that falls in is sold into the team bank |
| `EvacZone` | At the bottom of the shaft. Once the quota is met, a player who lands here leaves the run |

---

## Place the trap

1. Drag `BP_ExtractionTrap` into the level. Its pivot is the centre of the hatch at floor level.
2. Keep the space under it open. The trap brings its own floor slab and its `Chute`, and the `EvacZone` sits about 480 cm below the pivot: nothing may stop a falling player above that depth. In `L_Procedural`, the trap stands in the outliner folder `StartModule/Extraction`, with nothing built under it.
3. If your shaft is deeper or wider, select the zone in the Components panel and move it or change its `Box Extent`.

---

## Link a button

1. Drag `BP_InteractButton` into the level, next to the hatch but outside it.
2. Select its `BP_ActivationLinkComponent`.
3. Add an entry to `Targets` and pick the trap with the eyedropper.
4. Set `Type` on the button. The shipped map uses `Hold`, so a sale takes a deliberate press.

A lever works too, and so does anything that sends `BPI_Activatable` through a `BP_ActivationLinkComponent`. A lever turned on asks for a sale. Turned off, it closes the hatch. The link itself is explained on [Link a button or a lever to a door](../interaction/doors_buttons_and_levers.md).

After a sale, the hatch closes on its own. You never need a second press.

---

## The fields

All of them are on the placed `BP_ExtractionTrap`.

| Field | What it does | Default |
|---|---|---|
| `Open Angle`, in `Settings|Door` | How far the two leaves drop, in degrees | `90` |
| `Move Duration`, in `Settings|Door` | Seconds for the leaves to open or close | `0.8` |
| `Hold Open Time`, in `Settings|Extraction` | Seconds the hatch stays fully open after a sale, before it closes | `3` |
| `Detection Interval`, in `Settings|Extraction` | Seconds between two checks of the zones | `0.2` |
| `Detection Object Types`, in `Settings|Detection` | The collision object types the zones look for | `WorldDynamic`, `Pawn`, `PhysicsBody` |

---

## The sign

The sign is `WBP_ExtractionSign`, drawn on the trap's `SignDisplay` component. Its texts and colours are under `Settings|Style`: `Empty Text`, `Blocked Text`, `Ready Text`, `Evacuate Text`, and `Empty Color`, `Blocked Color`, `Ready Color`, `Evacuate Color`. Change them in the widget to change every trap.

---

## Floor arrows and signs

The start room of `L_Procedural` guides players to the trap with two Decal Actors, `Start_SignArrow_Hall` and `Start_SignLabel_Extraction`, in the outliner folder `StartModule/Signage`. They use `MI_Decal_ExtractionArrow` and `MI_Decal_ExtractionSign`.

A decal paints everything inside its box, loot included. Keep `Decal Size` X at `10` and sink the actor slightly behind the floor or wall. A thicker box paints the arrow on every item dropped on it.

---

## Two extraction points

Place a second `BP_ExtractionTrap` with its own shaft and its own button. Each trap has its own zones and its own sign. They sell into the same team bank, and both open for good when the quota is met.

---

## The dispatchers

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnZoneStatusChanged` | The trap's status changed. Gives the new `Status` | every player |

---

Next: [Change the quota, the run length and the recap](tune_the_run.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
