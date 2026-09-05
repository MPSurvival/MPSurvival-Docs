# Add a candidate

Anyone you can hire is one Data Asset. Four ship with the template, and they exist to show the four combinations of skills rather than to fill the list.

| Asset | Skills |
|---|---|
| `DA_Employee_Ana` | `[Cashier]` |
| `DA_Employee_Ben` | `[Stocker]` |
| `DA_Employee_Chloe` | `[Cashier, Stocker]` |
| `DA_Employee_Dmitri` | `[Stocker, Cashier]` |

The last two can do exactly the same work. They differ only in which job they reach for first, because the order of `Roles` is the priority.

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
| `DisplayName` | The name shown in the hiring list and the roster. A `Text` |
| `Portrait` | A `Texture2D`, drawn at 64 px in both lists |
| `Roles` | An array of `E_EmployeeRole`. The order is the priority |
| `WalkSpeed` | How fast they move |
| `WorkSpeed` | A divisor on how long a task takes. Higher is faster |
| `DailyWage` | Charged every day they work, with reason `Salary` |
| `CosmeticSet` | A `DA_CosmeticSet_*`, the wardrobe they are dressed from |

`WorkSpeed` does exactly one thing: `TaskSeconds / WorkSpeed`, clamped so it cannot be zero. There is no branch of code anywhere that behaves differently for a fast worker.

That is deliberate. Two stats that each do one measurable thing beat five stats where three do nothing, and the template used to have `Accuracy` and `Stamina` before they were cut for having no reader.

---

## The portrait

Draw it on a **dark** background, or at least draw it so it reads on one. The two staff apps use the same near-white ink as the rest of the interface, so a portrait that is white on white disappears.

64 px square is the size it is drawn at in both lists. Something a bit larger imports fine.

---

## Wardrobe

`CosmeticSet` works exactly as it does on a customer archetype. It points at a `BP_CosmeticSetDataAsset`, which is a list of slots, and each slot offers a list of skeletal meshes to pick from at random.

`DA_CosmeticSet_Employee` ships with the template so staff read as staff rather than as shoppers who wandered behind the counter.

If you add meshes, set their collision to none.

---

## The two apps

Both are apps on the office computer, so they come with the desktop rather than being separate screens.

**Hiring** lists everyone in `DA_App_Hiring.AvailableCandidates` with their portrait, name, skills, speed and daily wage, and a button that says `HIRE` or `ON STAFF`. Hiring is free at the moment of the click. The wage is what costs you.

**Staff** lists your roster. Each row shows the portrait, the name with `ON DUTY` or `OFF` beside it, the skills, seven day toggles, and start and end hours with `-` and `+` buttons.

Both pages end with `DAILY PAYROLL`, the sum of the wages of everyone on the roster.

Editing a shift edits `NextShift`. The change lands tomorrow morning, and the `ON DUTY` badge keeps showing today's truth in the meantime.

---

## Locking a candidate behind progress

A candidate is a `PrimaryDataAsset`, so it can be a reward on an unlock like anything else. `DA_Unlock_SeniorStaff` is the shipped example.

Put the candidate in an unlock's `Rewards` and they disappear from the hiring list until that unlock is granted. Leave them out of every unlock and they are available from day one. See [Unlocks](../progress/unlocks.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
