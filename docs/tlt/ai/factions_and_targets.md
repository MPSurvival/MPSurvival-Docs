# Factions and who fights who

A faction decides who an AI is willing to attack. Nothing in the chain that goes from perception to target choice to firing casts to the player character, so the player is not a special case: it is one more actor carrying a faction, exactly like an NPC or an infected.

Three things make the system:

- `Content/TheLastTemplate/Blueprints/DataAssets/Factions/BP_FactionDataAsset`, the Data Asset class.
- The faction assets in `Content/TheLastTemplate/Blueprints/DataAssets/Factions/Childs/`.
- `Content/TheLastTemplate/Blueprints/ActorComponents/BP_FactionComponent`, the component that puts a faction on an actor. It has one field.

---

## The three factions that ship

| Asset | `Faction Name` | `Hostile Factions` | Used by |
|---|---|---|---|
| `DA_Faction_Survivors` | `Survivors` | Hunters, Infected | the player |
| `DA_Faction_Hunters` | `Hunters` | Survivors, Infected | `DA_NPC_Roaming`, `DA_NPC_Melee_Roaming`, `DA_NPC_Path`, `DA_NPC_Stand` |
| `DA_Faction_Infected` | `Infected` | Survivors, Hunters | `DA_Zombie_Roaming` |

Everyone is hostile to everyone. No neutral faction and no allied faction ship with the template.

---

## What a faction asset holds

| Field | What it does |
|---|---|
| `Faction Name` | The name of the faction, as you read it in the Details panel. |
| `Hostile Factions` | The factions this one attacks. An array of faction assets. |

`Hostile Factions` is the only place hostility is written, and it is read as a plain list: a faction that is not in the array is not an enemy.

!!! warning
    The array is read one way only. Putting `DA_Faction_Hunters` in the hostile list of `DA_Faction_Survivors` makes survivors attack hunters. It does **not** make hunters attack survivors. You have to open the other asset and add the return line. Forget it and one camp opens fire while the other stands there doing nothing, with no error anywhere.

---

## Where an actor gets its faction

`BP_FactionComponent` carries a single field, `Faction`, under `Settings`. When something needs to know the faction of an actor, it looks in this order:

1. `Faction` on the actor's `BP_FactionComponent`, if you set one.
2. Otherwise, the `Faction` on the AI Data Asset that actor is running.

So the AI Data Asset is where you set the faction of a whole family of enemies, and the component is the override for one actor placed in a level.

Three classes ship with the component:

| Class | Where its faction comes from |
|---|---|
| `BP_PlayerCharacter` | `Faction` on the component, set to `DA_Faction_Survivors` |
| `BP_NPCCharacter` | the asset in `Data NPC` |
| `BP_ZombieCharacter` | the asset in `Data NPC` |

To change the player's side, open `BP_PlayerCharacter`, select `BP_FactionComponent` and change `Faction`. Set it to `DA_Faction_Hunters` and the hunters stop counting you as an enemy, while the infected still come for you. No graph is involved.

---

## No faction component means no AI ever targets you

Every other actor in a level, a prop, a pickup, a door, the training dummy, has no faction component. An AI never picks any of them as a combat target, and you do not have to filter them out anywhere.

The melee sweep reads factions the other way round. It refuses a candidate that resolves to the **same** faction as the attacker, so two infected clawing at you no longer hit each other. That check reads the faction, not the hostile list. An actor with no faction at all fails the same-faction test and stays hittable, which is why anyone can punch the training dummy.

---

## Choosing a target when two enemies are in view

An AI does not lock onto the first thing it sees. Every hostile actor it currently perceives is scored, and the best score wins. The comparison is redone every tick, so a target can change mid fight.

The score of a candidate is minus its distance in centimetres, plus the bonuses below. The bonuses are in centimetres too, so a bonus of 1500 means "treat this one as if it were 15 metres closer than it is".

These four fields live on the AI Data Asset, under `Settings > Perception > Targeting`.

| Field | What it does | Shipped |
|---|---|---|
| `Retaliation Window` | How long, in seconds, an actor that damaged this AI keeps the bonus below | `5` |
| `Retaliation Bonus` | Score added to whoever damaged this AI inside that window | `1500` |
| `Target Stickiness` | Score added to the target the AI already has, so it does not flip on a tie | `400` |
| `Lose Target Distance` | Past this distance the target is dropped | `3300`, and `3000` on `DA_Zombie_Roaming` |

Raise `Target Stickiness` for an enemy that commits to one victim and ignores the rest. Raise `Retaliation Bonus` for one that always turns on whoever just hurt it. Lower both towards zero and the AI simply attacks the nearest hostile.

---

## Make a faction fight itself

Nothing stops a faction from listing itself. Put `DA_Faction_Hunters` in the `Hostile Factions` of `DA_Faction_Hunters` and hunters start treating other hunters as enemies, which is enough to stage an infighting scene without a fourth asset.

Two rules still apply and they pull in opposite directions here: hostility comes from `Hostile Factions`, but the melee sweep refuses a same-faction candidate whatever that list says. A self hostile faction therefore fights itself at range and never lands a melee hit on its own members.

---

## Add a fourth faction

1. In `Content/TheLastTemplate/Blueprints/DataAssets/Factions/Childs/`, right click, then **Miscellaneous**, then **Data Asset**. Pick `BP_FactionDataAsset` in the class list. Name it, for example `DA_Faction_Raiders`.
2. Fill `Faction Name`.
3. Fill `Hostile Factions` with the factions your raiders attack.
4. Open each of those factions and add `DA_Faction_Raiders` to their own `Hostile Factions`. This is the step that is easy to skip.
5. Give the faction to an enemy. Either duplicate an AI Data Asset such as `DA_NPC_Roaming` and set its `Faction`, or select `BP_FactionComponent` on one placed actor and set `Faction` there.
6. Save.

The new faction works from that point on.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
