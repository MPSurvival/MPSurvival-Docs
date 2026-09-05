# Grand Opening

Welcome to the documentation for **Grand Opening**, a first person store management template for Unreal Engine.

You own a small shop. You order stock from a computer in the back office, the boxes arrive on the delivery bay, you carry them to the shelves and fill them one unit at a time. Customers come in, pick what they want, queue at your till, and you scan their items, take their money and count their change back by hand. At the end of the day you close the sign, read the report, and start again with whatever you made.

Everything in that loop is **single player** and built entirely with Blueprints.

Almost everything you will want to change is a **Data Asset** or a field in the **Details panel**, not a graph.

---

## How do I...

| I want to | Page |
|---|---|
| Add a product to the shop | [Add a product](stock/add_a_product.md) |
| Build a shelf of my own shape | [Build your own shelf](stock/make_your_own_shelf.md) |
| Add furniture the player can buy and place | [Add a placeable structure](build/add_a_structure.md) |
| Add a new kind of customer | [Add a customer archetype](customers/add_an_archetype.md) |
| Add someone to hire | [Add a candidate](staff/add_a_candidate.md) |
| Add an ad campaign | [Ad campaigns](progress/advertising.md) |
| Gate something behind progress | [Unlocks](progress/unlocks.md) |
| Write an app for the office computer | [Write your own app](computer/add_an_app.md) |
| Add a box size | [Add a box size](delivery/add_a_box_size.md) |
| Put a checkout in my own level | [Put a checkout in your level](checkout/place_a_checkout.md) |
| Build my own store level | [The three maps, and how a game starts](start/maps_and_startup.md) |
| Change a key | [The controls](start/controls.md) |
| Change the colours and fonts | [Colours, fonts and reskinning](ui/colours_and_fonts.md) |
| Add a job employees can do | [How employees work](staff/how_employees_work.md) |
| Add a footstep surface | [Surfaces, footsteps and sound](look/surfaces_and_sound.md) |

---

## Where to start

If you have just downloaded the template:

1. [Install and open the project](start/install_and_open.md)
2. [The three maps, and how a game starts](start/maps_and_startup.md)
3. [The controls](start/controls.md)

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
| [The office computer](computer/how_the_computer_works.md) | The desktop, the six apps, and writing an app of your own |
| [Advertising and progression](progress/advertising.md) | Campaigns, sidewalk panels, and the unlock catalogue |
| [Interface, menus and saving](ui/hud_and_prompts.md) | The HUD, the interaction prompt, world screens, the menus, the pause and the save slots |
| [Look and sound](look/surfaces_and_sound.md) | Physical surfaces, footsteps, the audio mix, the day/night cycle and the mesh kit |
| [Under the hood](architecture/the_rules.md) | The six rules to follow when you extend it, and where every asset lives |

---

## How it is played

- **Single player**, with a keyboard and a mouse.
- **Every displayed string is a `Text`**, so the widgets are yours to translate.
- **The keys live in `IMC_Gameplay`.** Dragging a mapping onto another key takes about five seconds.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
