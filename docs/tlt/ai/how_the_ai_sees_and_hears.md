# How the AI sees, hears and reacts

Every AI in the template, human or infected, runs the same chain:

1. An `AIPerception` component on the controller reports what it sees and what it hears.
2. `BP_AIBrain`, a component on the pawn, turns that into a target, a last known location and a combat mode.
3. The Behavior Tree reads the brain and decides what the AI actually does.

You never have to open the middle piece. The numbers you change sit at the two ends:

- The senses: `Content/TheLastTemplate/Blueprints/AI/BP_AIController`, on its `AIPerception` component.
- Everything the AI does about what it sensed: its AI Data Asset, in `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/`.

`BP_NPCController` and `BP_ZombieController` both derive from `BP_AIController` and change nothing about the senses. A hunter and a zombie see and hear exactly the same way. What separates them is the data asset.

---

## Sight

| Field on `AIPerception` | Ships at | What it does |
|---|---|---|
| `Sight Radius` | 1800 | How far the AI can notice you, in centimetres |
| `Lose Sight Radius` | 2400 | How far it keeps seeing you once it has you |
| `Peripheral Vision Half Angle Degrees` | 60 | Half the width of the vision cone |
| `Auto Success Range From Last Seen Location` | 400 | Inside this range of the last sighting, no clear line is needed |
| `Max Age` | 3 | Seconds before an unconfirmed sighting is forgotten |

Sight is a cone, not a sphere. At 60, the AI sees 60 degrees to each side of where it is facing, so a 120 degree window in front. Anything behind it does not exist, however close it is. This is why the direction an idle AI faces matters more than its position, and why `Idle Pattern` and stand points change how hard a place is to sneak through.

`Lose Sight Radius` is deliberately larger than `Sight Radius`. The AI notices you at 18 metres, then holds on to you out to 24. Without that gap, standing on the exact edge of the radius makes it flicker between seeing you and not.

`Auto Success Range From Last Seen Location` is the one that saves you from silly behaviour. Within 4 metres of where it last saw you, the AI does not need a clean line of sight. Without it, an AI standing next to a thin pillar loses its target the moment you step behind it.

---

## Hearing

| Field on `AIPerception` | Ships at | What it does |
|---|---|---|
| `Hearing Range` | 8000 | The furthest the AI can ever hear anything |
| `Max Age` | 1 | Seconds before a noise is forgotten |

`Hearing Range` is a ceiling, not the range of a noise. Every noise reports its own range at the moment it is made: a `BP_NoiseZone` ships with `Max Range` at 1600, a gunshot carries the noise range of the weapon that fired it. The AI hears the noise only if it is close enough for both numbers, so the smaller one is the one you feel. Raising `Hearing Range` on the controller does not make a quiet noise travel further.

See [Noise, and how to use it](noise_and_distractions.md) for what makes noise and how to make your own.

---

## Hearing never hands the AI a target

The brain builds its target list from the sight sense only. A noise cannot make an AI fight you.

What a noise does is write a last known location and push the AI into the `Search` combat mode. It walks to the spot and looks around. If you are still there when it arrives, sight picks you up and the fight starts from there.

That split is what makes stealth work. Throwing a bottle moves an AI, it does not aggravate it. If you want a sound to start a fight, put the AI somewhere the noise will pull it into its own vision cone.

---

## Losing you, and searching

| Field on the AI Data Asset | Ships at | What it does |
|---|---|---|
| `Lost Sight Grace` | 2 | Seconds the AI keeps a target after sight stops confirming it |
| `Search Give Up Time` | 8 | Seconds spent searching the last known location before giving up |
| `Lose Target Distance` | 3300 | Distance at which the target is dropped outright, 3000 on `DA_Zombie_Roaming` |

Break the line of sight and nothing happens for `Lost Sight Grace` seconds. That grace is what stops an AI from forgetting you every time you cross a doorway.

After it expires, the AI goes to the last place it had you and searches for `Search Give Up Time` seconds. A search is a walk to that point and a look around, not a spreading sweep of the building. It does not call other AI to the spot, and it does not remember where it searched. When the timer runs out it goes back to its idle pattern.

`Lose Target Distance` is the hard cut. Get more than 33 metres away and the target is dropped even if the AI can still see you.

---

## Getting shot when it cannot see you

| Field on the AI Data Asset | Ships at | What it does |
|---|---|---|
| `React To Damage` | true | Whether taking damage makes the AI investigate at all |
| `Alert Turn Time` | 1.5 | Seconds it spends turning towards the hit before it moves |
| `Damage Investigate Distance` | 900 | How far it is willing to travel towards where the hit came from |

Shoot an AI that has no idea you are there and it does not instantly know your position. It turns towards where the hit came from, taking `Alert Turn Time` to do it, then moves to investigate up to `Damage Investigate Distance` away. If your line of sight is clear by then it finds you and fights. If you moved, it searches and gives up like any other search.

Set `React To Damage` to false and the AI takes the hit without looking for you. Useful for something you want to be scenery until a script tells it otherwise.

---

## Choosing between several targets

When more than one hostile is visible, the brain scores them and keeps the best one. Two fields bend that score.

| Field on the AI Data Asset | Ships at | What it does |
|---|---|---|
| `Target Stickiness` | 400 | Bonus for the target the AI already has |
| `Retaliation Window` | 5 | Seconds after being damaged that the attacker gets a bonus |
| `Retaliation Bonus` | 1500 | Size of that bonus |

`Target Stickiness` stops the AI from swapping every time someone else becomes marginally closer. `Retaliation Bonus` is much larger, so for five seconds after being hit, the AI will almost always turn on whoever hit it. Lower it if you want an AI that ignores chip damage and stays on its original target.

The senses themselves do not care about teams. `Detection By Affiliation` is fully ticked on both sight and hearing, so the perception component reports friends, neutrals and enemies alike. Deciding who is a valid target is a separate faction check. See [Factions and who fights who](factions_and_targets.md).

---

## Turning a sense off

`Use Sight` and `Use Hearing` on the AI Data Asset switch a sense off entirely. Both ship true on every preset. The controller applies them when it possesses the pawn, so the switch belongs to the data asset and covers every AI using it. If you want one blind guard, give it its own data asset.

!!! warning "An AI with `Use Sight` off can never fight"
    Targets only ever come from the sight sense. Turn sight off and the AI will still hear noises, walk to them and react to damage, but it will never acquire a target and never attack. Use it for a genuinely blind enemy, not as a way to keep an AI passive until you wake it up.

---

## Where each number lives

| You want to change | Open |
|---|---|
| How far, how wide, how long the AI perceives | `BP_AIController`, the `AIPerception` component |
| How long it holds a target, how long it searches, how it reacts to damage, how it picks between targets | The AI Data Asset in `DataAssets/AI/Childs/` |
| Which AI is hostile to which | The faction asset referenced by `Faction` on the data asset |
| What it does with no target | `Idle Pattern` and the path, stand point or roam fields on the data asset |

The split matters when you tune. Anything on the controller is shared by every AI in the project, because both controllers inherit it and nothing overrides it per pawn. Anything on a data asset is per AI type, and making a new type is duplicating a data asset. If you want one sharp eyed sniper and one short sighted brute, that difference cannot come from `Sight Radius`, so build it out of the data asset fields instead, or give the sniper its own controller class.

---

## When it has nothing to chase

With no target and no noise, the AI falls back to `Idle Pattern`: `None`, `Path`, `Roaming` or `StandPoint`. That is where it spends most of its life, and it is where the player sees it before anything starts.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
