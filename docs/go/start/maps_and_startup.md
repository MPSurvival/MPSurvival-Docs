# The three maps, and how a game starts

The template ships three levels. They do different jobs, and only one of them is the game.

| Map | What it is |
|---|---|
| `L_MainMenu` | The menu. A street scene with no gameplay in it, a slow camera drift, and the menu widgets on top. |
| `L_ExampleMap` | The store. This is the level a game is played in: sales floor, back office, delivery bay and the street outside. |
| `L_ShowcaseMap` | The demo hub. Fourteen white rooms in a row, one per system, each with a sign on the far wall and the Blueprints of that system laid out along the walls. |

`L_ExampleMap` is the project's editor startup map, so opening the project drops you straight in the shop.

---

## The startup chain

```
L_MainMenu  →  BP_MenuGameMode  →  BP_MenuPlayerController
                                     ↓  NEW GAME / LOAD
L_ExampleMap  →  BP_StoreGameMode  →  BP_StorePlayerController  →  BP_StoreCharacter
```

`BP_StoreGameMode` is the project's default game mode, set in **Project Settings → Maps & Modes**. `L_MainMenu` overrides it with `BP_MenuGameMode` in its World Settings.

The level the menu opens is not written into a graph. It is `GameLevel`, a soft level reference on `BP_MenuPlayerController` in the `Settings|Menu` category. Point it at your own level and the whole menu follows.

---

## What runs the game

Four objects hold everything, and each of them owns one kind of thing.

| Object | What it owns |
|---|---|
| `BP_StoreGameInstance` | The save slots. It is the only thing in the project that talks to disk, and it survives a level change. |
| `BP_StoreGameState` | The six managers, as components. Money, time, deliveries, staff, economy, ads, progression. |
| `BP_StoreGameMode` | The `PlayerStart`, and `CleanForNextDay()`, which resets the world between two days. |
| `BP_StorePlayerController` | The HUD, the pause menu, and the flow that closes the store. |

The managers all sit on the **game state**, not on the game instance, so a level change starts a clean game.

| Component | What it does |
|---|---|
| `BP_StoreManager` | Balance, transaction journal, store rating, day, clock, task queue |
| `BP_DeliveryManager` | Pending orders and the boxes that arrive from them |
| `BP_StaffManager` | The roster, hiring, shifts and wages |
| `BP_EconomyManager` | Daily charges and the wholesale market |
| `BP_AdManager` | Running campaigns and the panels that show them |
| `BP_ProgressionManager` | The unlock catalogue and what has been granted |

---

## What a level needs to be playable

If you build your own store level rather than editing `L_ExampleMap`, it needs these actors. Miss one and the symptom is usually silence, not an error.

| Actor | Why |
|---|---|
| `PlayerStart` | Where you spawn, and where you are teleported back to at the start of each day |
| `BP_CustomerSpawner` | Customers are born here |
| `BP_CustomerExit` | Where they walk to when they leave. Without it they walk to the spawner instead |
| `BP_DeliveryBay` | Where ordered boxes land. It registers itself with the delivery manager at `BeginPlay` |
| `BP_StaffEntrance` | Where employees appear at the start of their shift |
| `BP_StoreZoneVolume` | At least one, or nothing can be placed anywhere. See [Zones](../build/zones.md) |
| `NavMeshBoundsVolume` | It has to cover the spawner, the exit and the whole sales floor |
| At least one `BP_Checkout` with its scanner | No usable checkout means no customer is even born. See [How the checkout works](../checkout/how_checkout_works.md) |
| `BP_DayNightCycle` | Optional, but without it the sun never moves |

The customer exit has to sit **inside** the nav mesh volume. `Move To` will not project a destination that falls outside it, and the customer stands there instead of leaving.

---

## The front door is a one-way lock

`BP_EntranceDoor` hands out the doorway in one direction at a time, and anyone travelling the other way holds position until it clears. Without that, two AI walking into the same doorway from opposite sides push the same door leaf and neither gives way, because their capsules ignore each other.

**If you build your own front door, duplicate `BP_EntranceDoor`**: the lock lives there. `BP_OfficeDoor` does not have it, which is fine as long as no customer goes through it.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
