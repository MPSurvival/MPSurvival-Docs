# Grand Opening

Welcome to the documentation for **Grand Opening**, a first person store management template for Unreal Engine.

You own a small shop. You order stock from a computer in the back office, the boxes arrive on the delivery bay, you carry them to the shelves and fill them one unit at a time. Customers come in, pick what they want, queue at your till, and you scan their items, take their money and count their change back by hand. At the end of the day you close the sign, read the report, and start again with whatever you made.

Everything in that loop is **single player** and built entirely with Blueprints.

The idea behind the project is the same one behind every page here: the thing you want to change is almost always a **Data Asset** or a field in the **Details panel**, not a graph. A new product, a new customer type, a new piece of furniture, a new employee, a new ad campaign, a new app on the office computer. Each one is an asset you fill in.

---

## Where to start

If you have just downloaded the template:

1. [Install and open the project](start/install_and_open.md)
2. [The three maps, and how a game starts](start/maps_and_startup.md)
3. [The controls](start/controls.md)

After that, go straight to the system you care about. Every chapter opens with a page that explains how the system is put together, and the pages after it each answer one question, such as "how do I add a product" or "how do I make my own shelf".

---

## The chapters

| Chapter | What it covers |
|---|---|
| [Getting started](start/install_and_open.md) | Opening the project, the maps, the game modes and the keys |
| [The store day](store/how_a_day_works.md) | The clock, closing time, the daily report, the balance, the store rating, rent, tax and the wholesale market |
| [Products and shelves](stock/how_storage_works.md) | Spline storage rows, product footprints, shelf types, and building a shelf of your own shape |
| [Deliveries and the back room](delivery/orders_and_boxes.md) | Ordering, the delivery bay, carrying and unpacking boxes, and the storage rack |
| [The checkout](checkout/how_checkout_works.md) | The queue, the belt, the barcode scanner, cash, change, the card terminal and the till |
| [Customers](customers/how_a_customer_thinks.md) | Archetypes, the shopping loop, patience, traffic per hour and the spawn rules |
| [Employees](staff/how_employees_work.md) | Hiring, shifts, wages, and the task queue the AI pulls from |
| [Building the store](build/how_placement_works.md) | Buying furniture, the build menu, placing on floors, walls and ceilings, edit mode and zones |
| [The office computer](computer/how_the_computer_works.md) | The diegetic desktop, the six apps, and writing an app of your own |
| [Advertising and progression](progress/advertising.md) | Campaigns, sidewalk panels, and the unlock catalogue |
| [Interface, menus and saving](ui/hud_and_prompts.md) | The HUD, the interaction prompt, world screens, the menus, the pause and the save slots |
| [Look and sound](look/surfaces_and_sound.md) | Physical surfaces, footsteps, the audio mix, the day/night cycle and the mesh kit |
| [Under the hood](architecture/the_rules.md) | The six ownership rules the whole template is built on, and where every asset lives |

---

## What this template does not do

Said here so you find out now rather than three hours in.

- **No multiplayer.** Nothing in the project is replicated, on purpose. The co-op version is a separate product.
- **No options screen and no key rebinding.** The keys are the ones in `IMC_Gameplay`.
- **No shipped translation.** Every displayed string is a `Text`, so you can translate it yourself, but there is no localisation pipeline in the box.
- **No gamepad support.** The game is played with a keyboard and a mouse.
- **No discounts, no shoplifting, no store expansion.** These were considered and dropped.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
