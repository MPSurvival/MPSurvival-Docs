# Add a candidate

Anyone you can hire is one Data Asset. Four ship with the template, one per combination of skills:

| Asset | Skills |
|---|---|
| `DA_Employee_Ana` | `[Cashier]` |
| `DA_Employee_Ben` | `[Stocker]` |
| `DA_Employee_Chloe` | `[Cashier, Stocker]` |
| `DA_Employee_Dmitri` | `[Stocker, Cashier]` |

The last two do the same work and differ only in which job they reach for first, because the order of `Roles` is the priority.

---

## The recipe

1. Right click in `Blueprints/DataAssets/Employees/Childs/` → **Data Asset** → `BP_EmployeeCandidateDataAsset`.
2. Name it `DA_Employee_<Name>`.
3. Fill it in.
4. Add it to `AvailableCandidates` on `DA_App_Hiring`, or nobody can hire them.

---

## The fields

| Field | What it does |
|---|---|
| `DisplayName` | The name in the hiring list and the roster |
| `Portrait` | A `Texture2D`, drawn at 64 px in both lists |
| `Roles` | An array of `E_EmployeeRole`. The order is the priority |
| `WalkSpeed` | How fast they move |
| `WorkSpeed` | A divisor on how long a task takes. Higher is faster |
| `DailyWage` | Charged every day they work |
| `CosmeticSet` | A `DA_CosmeticSet_*`, the wardrobe they are dressed from |

---

## The portrait

Draw it on a **dark** background, or at least so it reads on one: the staff apps use the same near-white ink as the rest of the interface, and a white-on-white portrait disappears. 64 px square is the drawn size; something larger imports fine.

---

## Wardrobe

`CosmeticSet` works as it does on a customer archetype: a list of slots, one option per slot drawn at random. `DA_CosmeticSet_Employee` ships so that staff read as staff rather than as shoppers who wandered behind the counter.

If you add meshes, set their collision to none.

---

## The two apps

**Hiring** lists everyone in `DA_App_Hiring.AvailableCandidates` with their portrait, name, skills, speed and daily wage, and a `HIRE` button. Hiring costs nothing at the click; the wage is what costs you.

**Staff** lists your roster: portrait, name with `ON DUTY` or `OFF`, skills, seven day toggles, and start and end hours. Editing a shift there takes effect tomorrow morning, and the `ON DUTY` badge keeps showing today's truth in the meantime.

Both pages end with `DAILY PAYROLL`, the sum of the wages on the roster.

---

## Locking a candidate behind progress

Put the candidate in the `Rewards` of an unlock and they stay out of the hiring list until it is granted. `DA_Unlock_SeniorStaff` is the shipped example. Leave them out of every unlock and they are available from day one. See [Unlocks](../progress/unlocks.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
