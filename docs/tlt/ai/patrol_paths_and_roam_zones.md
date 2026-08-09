# Patrol paths, roam zones and stand points

An AI with nothing to fight is not idle. It runs one of four movement patterns, chosen by the `Idle Pattern` field on its Data Asset. Three of those patterns need an actor placed in your level to tell the AI where to go.

This page covers those actors and how the AI reads them. The fields themselves are listed on [The AI settings asset](ai_settings_data_asset.md).

The markers live in `Content/TheLastTemplate/Blueprints/Environments/AI/`.

---

## The four patterns

| `Idle Pattern` | What the AI does | What you place |
|---|---|---|
| `None` | Stands where you dropped it and never moves on its own | nothing |
| `Path` | Walks a loop of markers, in order | several `BP_AIPath` |
| `Roaming` | Picks random reachable points around an area | one `BP_AIRoamZone`, or nothing |
| `StandPoint` | Holds one spot and walks back to it | one `BP_AIStandPoint` |

The pattern only runs when the AI has no target. As soon as it sees or hears something it drops the pattern and fights, then returns to it. That side of things is on [How the AI sees, hears and reacts](how_the_ai_sees_and_hears.md).

All three markers are matched by name, not by reference. You type a string on the actor and the same string on the Data Asset, and that is the link. Nothing points at anything, so you can add and delete markers freely. The match ignores upper and lower case.

Note that the field names differ slightly on the two sides:

| On the Data Asset | On the actor in the level |
|---|---|
| `Path Id` | `BP_AIPath` → `Path Id` |
| `Roam Zone Id` | `BP_AIRoamZone` → `Zone Id` |
| `Stand Point Id` | `BP_AIStandPoint` → `Point Id` |

---

## Pattern None

Set `Idle Pattern` to `None` and the AI stops moving on its own. It still sees, hears, fights and dies normally. Use it for an enemy you placed exactly where you want it, behind a counter or at a window, that should never wander off the mark.

---

## Pattern Path: a patrol loop

1. Drag a `BP_AIPath` into the level at the first waypoint.
2. Set `Path Id` to a name of your choice, for example `Guard_Loop`.
3. Set `Path Index` to `0`.
4. Copy it to each of the other waypoints, keeping the same `Path Id` and giving `Path Index` the values `1`, `2`, `3` and so on.
5. Open your AI Data Asset, set `Idle Pattern` to `Path` and `Path Id` to `Guard_Loop`.

The AI starts at index 0 and walks the markers in numeric order. When it arrives it waits, then goes to the next one. When there is no marker at the next index it goes back to 0, so the patrol is always a loop. There is no back and forth mode.

That wrap rule is also the trap: the indexes have to start at 0 and run without a gap. If you number four markers `0`, `1`, `3`, `4` the AI walks 0 and 1, finds nothing at 2, and restarts. Two markers with the same `Path Id` and the same `Path Index` are worse, because you cannot predict which one wins.

The AI counts as arrived when it is within 120 cm of the waypoint on the ground plane, so you do not need to place them precisely.

`DA_NPC_Path` ships as a working example, with `Path Id` set to `Path_01`.

---

## Pattern Roaming, with a zone

1. Drag a `BP_AIRoamZone` into the level at the centre of the area you want covered.
2. Set `Zone Id`, for example `Yard`.
3. Select the `Zone Sphere` component and set `Sphere Radius` to the size of the area. The sphere you see in the viewport is the roam area.
4. On the Data Asset, set `Idle Pattern` to `Roaming` and `Roam Zone Id` to `Yard`.

The AI then picks random points on the navmesh inside the sphere, walks to one, waits, and picks another. It does not need to start inside the zone.

Two things filter each pick:

- `Roam Min Radius`, `300` by default, is the shortest move allowed. A candidate closer than that to the AI is thrown away and a new one is tried on the next frame. This is what stops an AI from shuffling on the spot.
- `Roam Separation`, `600` by default, is the space kept from other AI. A candidate is rejected if another AI is standing that close to it, or has already reserved a destination that close to it.

!!! warning
    If `Roam Min Radius` is larger than the area the AI can reach, no candidate ever passes the filter and the AI stands still forever, with no error. That happens when you shrink a zone below 300 cm, or raise `Roam Min Radius` above `Roam Max Radius` without checking. When a roaming AI refuses to move, this is the first thing to look at.

---

## Pattern Roaming, with no zone

Leave `Roam Zone Id` empty and you do not have to place anything. The AI roams around the spot where it stands, out to `Roam Max Radius`, `1200` cm by default. `Roam Min Radius` still applies.

This is what the shipped `DA_NPC_Roaming` and `DA_Zombie_Roaming` do. It is the fastest way to get a level populated: drop enemies where you want them and they stay in their own neighbourhood.

!!! tip
    The AI decides its roam area once, the first time it needs to move, and keeps it for the rest of the game. Moving a zone actor at runtime does not move the AI's area. In the editor it behaves exactly as you would expect, so this only matters if you move zones with a graph.

---

## Pattern StandPoint: hold a position

1. Drag a `BP_AIStandPoint` into the level.
2. Set `Point Id`, for example `Door_Watch`.
3. On the Data Asset, set `Idle Pattern` to `StandPoint` and `Stand Point Id` to `Door_Watch`.

The AI walks to the point and stays there. If anything pushes it more than 150 cm away, a fight, a hit reaction, a shove from another character, it walks back as soon as it is free again. This is the pattern to use for a guard that must return to its post after chasing you.

Only the location of the marker is used. Its rotation is ignored, so you cannot use it to choose which way the AI faces while it waits.

`Move Delay` and `Move Delay Jitter` are `0` on `DA_NPC_Stand`, because a stand point AI has nothing to wait for.

---

## Setting the pace

`Path` and `Roaming` both pause at each destination before choosing the next one. The wait is `Move Delay` seconds plus a random amount between zero and `Move Delay Jitter`.

The defaults are `2` and `3`, so an AI stops for two to five seconds at every stop. Raise `Move Delay` for a slow, bored patrol. Set both to `0` for an enemy that never stops walking. The jitter is not decoration: it is what stops a group of AI sharing a Data Asset from moving in lockstep.

---

## The ids are read from the whole level

When an AI looks for its path, its zone or its stand point, it searches every matching actor in the level, not only nearby ones. Two patrols in two different buildings that both use `Path_01` are one patrol, and the AI will happily walk across the map between them.

Give every route its own name. `Path_Warehouse`, `Path_Roof`, `Stand_Gate` cost nothing and remove the whole class of problem.

The second half of the same rule: the id lives on the Data Asset, not on the pawn. Every AI pointed at the same Data Asset looks for the same id. If you want two separate patrol routes, you need two Data Assets. Duplicate `DA_NPC_Path` in `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/`, change `Path Id`, and point the second group at the copy.

---

## Three guards on one loop

1. Place four `BP_AIPath` around the route, all with `Path Id` set to `Guard_Loop` and `Path Index` `0` to `3`.
2. Duplicate `DA_NPC_Path` and name the copy, for example `DA_NPC_GuardLoop`.
3. In the copy, set `Path Id` to `Guard_Loop`. `Idle Pattern` is already `Path`.
4. Drag three `BP_NPCCharacter` into the level and set `Data NPC` to `DA_NPC_GuardLoop` on each.
5. Make sure a `NavMeshBoundsVolume` covers the whole route.

All three start at index 0, so they leave together and bunch up on the first marker. The random part of `Move Delay Jitter` pulls them apart over the next few stops. If you want them staggered from the first second, place them at different points of the route, or give each one its own path.

Patrol has no separation rule. `Roam Separation` only applies to `Roaming`, so guards on a path will walk through each other at corners. Keep the loop wide if that bothers you.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
