# How AI combat is put together

Four pieces make an enemy fight. Each one answers a different question, and knowing which one owns what is how you find where to type a number.

| Piece | Where it is | The question it answers |
|---|---|---|
| `BP_AIController` | the controller possessing the pawn | what the AI sees and hears |
| `BP_AIBrain` | a component on the pawn | who the target is, and what mode the AI is in |
| A Behavior Tree | an asset named on the AI Data Asset | what the AI does about it, right now |
| `BP_CombatComponent` | a component on the pawn | what an attack actually is |

One sentence carries most of it: **the tree decides when to attack, the component decides what an attack is and whether it is allowed.** Almost every tuning question turns out to be a question of which of those two owns the value.

---

## The controller

`Content/TheLastTemplate/Blueprints/AI/BP_AIController` carries the `AIPerception` component and runs the tree. Two children ship, one per pawn:

- `BP_NPCController`, used by `BP_NPCCharacter`
- `BP_ZombieController`, used by `BP_ZombieCharacter`

Neither child declares a variable of its own. Both are already set as `AI Controller Class` on their pawn, so placing an enemy never involves assigning a controller.

The senses are the part of the controller you will actually open. They are covered in [How the AI sees, hears and reacts](how_the_ai_sees_and_hears.md).

---

## The brain

`Content/TheLastTemplate/Blueprints/ActorComponents/BP_AIBrain` sits on the pawn. It takes what the perception component reported and turns it into three things the tree can use: a target, a last known location, and a combat mode. The mode is one of `Idle`, `Search`, `Ranged` or `Melee`.

The brain is also the only thing in the project that decides where an AI looks and whether its body turns to follow. If you write a task of your own, do not set a focus in it.

There is nothing to tune here. Nearly every field on the brain is runtime state, written while the game runs. Its settings are the AI Data Asset the pawn points at, and that asset is listed field by field at the end of this page.

---

## The behavior tree

Two trees ship:

- `Content/TheLastTemplate/Blueprints/AI/NPC/Behavior/BT_NPC`
- `Content/TheLastTemplate/Blueprints/AI/Zombie/Behavior/BT_Zombie`

Which one an AI runs is the `Behavior Tree` field on its AI Data Asset. The controller runs whatever the asset names. That makes the tree a per preset choice and not a per class one: point a human preset at `BT_Zombie` and that human runs the infected tree.

The tasks each tree uses live in a `Tasks` folder next to it.

| Tree | Task | What it is for |
|---|---|---|
| `BT_NPC` | `BTS_CombatSense` | A service. It keeps the tree's picture of the fight current from the brain, every tick. It is also what makes the AI consider a dodge when a blow comes at it. |
| `BT_NPC` | `BTT_IdleBehaviour` | Runs the `Idle Pattern` when there is no target |
| `BT_NPC` | `BTT_MoveToLastKnown` | Walks to the last known location and searches it |
| `BT_NPC` | `BTT_ChaseTarget` | Closes the distance to the target |
| `BT_NPC` | `BTT_CombatStrafe` | Moves side to side while shooting |
| `BT_NPC` | `BTT_ShootBurst` | Fires a burst |
| `BT_NPC` | `BTT_MeleeEngage` | Walks in until `Strike Range` minus `Strike Stop Margin`, stops, and asks the combat component for an attack |
| `BT_NPC` | `BTT_MeleeCircle` | Present, but does almost nothing today. See the last section. |
| `BT_NPC` | `BTT_MeleeRetreat` | Empty, and never reached. See the last section. |
| `BT_Zombie` | `BTS_ZombieSense` | The infected service, same job as `BTS_CombatSense` |
| `BT_Zombie` | `BTT_ZombieIdle` | Runs the `Idle Pattern` when there is no target |
| `BT_Zombie` | `BTT_ZombieChase` | Chases the target and asks for an attack on arrival |

Both trees share one blackboard, `Content/TheLastTemplate/Blueprints/AI/BB_AI`, and the service on each tree is what fills it. You do not need to open it to tune an AI. If you add a task of your own, read from the blackboard rather than reading the world a second way, so that your task agrees with everything already there.

---

## The combat component

`BP_CombatComponent` is the same component on the player, on the human and on the infected. It owns the montage that plays, the sweep that finds a hit, the damage, the spacing between fighters, the pause between blows, the dodge and the stealth finisher. What separates the three characters is the numbers in its Details panel, not the graph. All of that is in [How melee combat works](../melee/melee_how_it_works.md).

Two things about it are worth knowing on the AI side.

The melee sweep does not aim at a designated victim. It takes the first character it touches, and it refuses a character of the same faction. That is why two infected mauling you at the same time cannot hit each other.

The infected's attack animation does not come from its AI Data Asset. `Attack Montage` is empty on `DA_Zombie_Roaming` and the infected still attacks, because the montages come from `Attack Montages` on its `BP_CombatComponent`. Adding an animation to an enemy is a component field, never a data asset field.

---

## Where a setting goes

| You want to change | Open |
|---|---|
| How far, how wide and how long the AI perceives | `BP_AIController`, its `AIPerception` component |
| Who it is willing to attack | The faction asset named by `Faction` on the AI Data Asset |
| Which tree it runs | `Behavior Tree` on the AI Data Asset |
| What it does with no target | `Idle Pattern` and the id fields on the AI Data Asset |
| When it enters melee, how close it walks, how often it strikes, how it strafes | The AI Data Asset |
| The gun it spawns with, and its spare rounds | `Default Weapon Class` and `Starting Reserve Ammo` on the AI Data Asset |
| What an attack looks like, how far it reaches and how much it hurts | `BP_CombatComponent`, in the pawn's Details |
| Whether it can dodge, and whether it can be finished | `BP_CombatComponent`, in the pawn's Details |

The short version: anything about **going somewhere** is on the AI Data Asset, anything about **the blow itself** is on the combat component. Nothing on that list is a graph.

!!! warning "The two sides share one scale and must agree"
    The data asset decides where the AI stops walking, `Strike Range`. The component decides how far a blow is allowed to be requested, `Max Attack Range`. Raise `Strike Range` past `Max Attack Range` and the AI will walk to a spot where its own attack is refused, then stand there doing nothing, with no error anywhere. Change one and check the other. The full scale of five distances is on the melee page linked just above.

---

## What this system does not do

Said plainly so you do not spend an evening looking for it.

- **Enemies do not take positions around you.** They close in and attack. Nobody circles, nobody backs off to let another one through. `BTT_MeleeCircle` carries only enough to keep the tree from spinning, and `BTT_MeleeRetreat` is an empty task that nothing ever selects. What keeps a crowd from all swinging at once is `Use Attack Hold` and `Attack Hold Time` on each attacker's own component, not a coordinator.
- **An AI that finds you does not call the others.** There is no shared alert, no group state, no squad. Two enemies reacting together is two enemies that both sensed you.
- **A noise never hands an AI a target.** It moves the AI and starts a search. See [Noise, and how to use it](noise_and_distractions.md).
- **There is no difficulty setting that scales the AI.** An easier or harder enemy is a duplicated Data Asset with different numbers.

To watch the whole thing run, tick `Log Combat State` on a character's `BP_CombatComponent`. Its state is printed as it changes, which is the fastest way to tell whether the tree never asked for an attack or the component refused one.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
