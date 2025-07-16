# 🚀 How to create a wieldable class ?

### 1. Choose your base blueprint class

There are two main base blueprints you can use:

- `BP_WieldableMaster` for general wieldable items (guns, tools, etc.)
- `BP_WieldableMelee` if you want to create a melee weapon with built-in melee logic

If you want a firearm, create a child from `BP_WieldableMaster` and customize it.

---

### 2. Configure fields in `BP_WieldableMaster`

Open your wieldable blueprint and set the following:

| Property           | Description                                                      |
|--------------------|------------------------------------------------------------------|
| `Durability Consumer` | Handles item durability usage                                   |
| `Equip Animation FPS` | Animation played when equipping in first-person view           |
| `Equip Animation TPS` | Animation played when equipping in third-person view           |

---

### 3. Configure fields in `BP_WieldableMelee`

If your wieldable is melee-based, configure these extra fields:

| Property            | Description                                                    |
|---------------------|----------------------------------------------------------------|
| `Base Damage`       | Damage dealt per melee hit                                      |
| `Melee Box`         | Collision box used for hit detection                            |
| `Melee Range`       | Maximum range of melee attack                                  |
| `Melee Cooldown`    | Time between melee attacks                                      |
| `Melee Hit Delay`   | Delay after attack before hit detection (trace) happens       |
| `Attack Animation FPS` | First-person attack animation                                |
| `Attack Animation TPS` | Third-person attack animation                                |
| `Hit Shake Force`   | Camera shake intensity on hit                                   |
| `Swing Shake Force` | Camera shake intensity on swing                                 |
| `Correct Hit Sound` | Sound played when hitting a valid target                        |
| `Attack Sound`      | Sound played on melee attack                                    |
| `Gatherable Multiplier` | Multiplier affecting gatherable loots                        |

---

### 4. Customize and extend

- Add new animations or sounds to suit your weapon style.
- Use the existing logic in the base blueprints as a foundation.
- For firearms, extend `BP_WieldableMaster` and add your shooting logic.

!!! tip  
    Keep your wieldable blueprints modular and reusable for easy future updates.
