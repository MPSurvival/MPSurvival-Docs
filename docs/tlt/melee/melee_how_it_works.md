# How melee combat works

Melee is one component, `BP_CombatComponent`, and the same component sits on the player, on the human NPC and on the infected. It owns the attack cycle, the strike sweep, the spacing between fighters, the knockback, the dodge and the stealth finisher.

What changes between the three characters is the numbers in its Details panel. Not the graph.

- The component: `Content/TheLastTemplate/Blueprints/ActorComponents/BP_CombatComponent`
- The strike window: `Content/TheLastTemplate/Animations/AnimStates/ANS_MeleeStrike`
- The attack animations: `Content/TheLastTemplate/Animations/Montages/Characters/Combat/`

You never open the component's graph. Everything you change is a field in a Details panel, an entry in an array, or the position of a notify on a montage timeline.

---

## Where the settings live

Select `BP_CombatComponent` in the **Components** list of a character, and the whole melee setup for that character is in one Details panel. The three characters that carry it are:

- `Content/TheLastTemplate/Blueprints/PlayerCharacter/BP_PlayerCharacter`
- `Content/TheLastTemplate/Blueprints/AI/NPC/BP_NPCCharacter`
- `Content/TheLastTemplate/Blueprints/AI/Zombie/BP_ZombieCharacter`

Read the values on the character, not on the component asset. The component's own class defaults ship with `Attack Montages`, `Dodge Montages` and `Stealth Finishers` all empty, so nothing on the bare component fights. Each character fills them in.

---

## What starts an attack

The player punches with the **left mouse button** while the hands are empty. That is not an Input Action, so it is not in the rebinding list. See [Change the controls](../start/change_the_controls.md).

An AI asks for the attack itself once its behavior tree has walked it to `Strike Range`. That range, the cooldown between attacks and the number of blows in a combo all live on the AI Data Asset, not on the combat component. See [Tune a melee AI](../ai/tune_melee_ai.md).

Both go through the same request, and it is refused in the same three cases:

- the character is already inside an attack, in windup, strike or recovery
- the character it is fighting is mid attack. This is what makes an exchange alternate instead of both landing at the same instant.
- `Use Attack Hold` is on and the hold has not run out yet

When it is allowed, one montage is picked at random from `Attack Montages` and played. After the montage blends out, the character waits `Recovery Delay` before it can act again.

---

## What decides when the strike lands

A moving hand does nothing on its own. Damage is only possible inside the window of an `ANS_MeleeStrike` notify state placed on the montage timeline. Where you drag that window is the timing of the hit.

The notify carries three fields:

| Field on the notify | What it does | Value on a fresh notify |
|---|---|---|
| `Hand Bone` | Which bone the sweep follows | `Hand_R` |
| `Hit Radius` | Radius of the sweep, in centimetres | `45` |
| `Target Region` | Which part of the victim the animation is aiming at, from `E_HitRegion` | `Head` |

Inside the window, the component sweeps a sphere along the hand from one frame to the next. `Use Both Hands` on the component overrides the notify's `Hand Bone` and sweeps `Hand Bone Left` and `Hand Bone Right` both, so a left hook is measured on the hand that actually moves. The radius used is the larger of the notify's `Hit Radius` and the component's `Default Hit Radius`, so an old notify can never shrink your reach. One swing lands one hit.

!!! warning
    `Target Region` is `Head` on any notify you place and do not touch. The blow still lands, but the physical reaction starts from the head, so a body shot will read wrong. Set it on every notify you place.

The sweep does not pick a designated victim. It takes the first character in range that can take a hit, which has three consequences worth knowing before you add your own characters:

- the victim must be a **Character** carrying a `BP_HitReactionManager`. Your own pawn will simply not be hit by melee without one.
- a character of the **same faction** is never hit, so two infected mobbing you do not punch each other
- an actor with **no faction** is always hittable. That is how `BP_TrainingDummy` takes a punch.

Because nobody is designated, a human NPC and an infected fighting side by side can hit each other for real.

---

## What happens the moment a hit connects

Everything below fires at the one point where contact is proven, so it is identical whether the player is punching or being punched:

| What | Field | Shipped |
|---|---|---|
| Damage | `Strike Damage` | `10` on the player and the NPC, `12` on the infected |
| Physical reaction | `Melee Hit Profile` on `BP_HitReactionManager` | `DA_Hit_Punch` |
| Push on the victim | `Knockback Speed` | `250` |
| Direction of that push | `Impulse Along Aim` | on: the push runs attacker to victim, flattened, not along the instant hand direction |
| How far off centre it is applied | `Impact Offset` | `10`, which is what makes the victim turn rather than only slide |
| Sound at the impact bone | `Impact Sound` | `SC_Punch` |
| Screen shake | `Hit Shake Scale` | `1` |
| Hit stop | `Use Hit Stop`, `Hit Stop Scale`, `Hit Stop Time` | on, `0.35`, `0.09` seconds |

Hit stop and the full strength shake only arm when the player is one of the two fighters. A brawl between two AI across the street does not slow your game or shake your screen, it only sends the distance attenuated world shake.

The damage itself goes through the normal damage path, and the reaction through the hit profile. Both are covered in [How damage works](../damage/how_damage_works.md) and [Tune hit reactions](../damage/tune_hit_reactions.md).

---

## The distance scale

Five distances decide how a fight sits. They are shipped in this order on purpose, and changing one without looking at its neighbours is the usual way to break the feel:

| Distance | Field | Shipped |
|---|---|---|
| The lunge inside the strike itself | `Strike Reach` on the component | `80` |
| How close two fighters let each other stand | `Min Combat Distance` on the component | `100` |
| Where the attacker settles during the windup | `Align Distance` on the component | `115` |
| Where the AI stops walking | `Strike Range` on the AI Data Asset | `130` human NPC, `100` infected |
| Past this, an attack request is refused | `Max Attack Range` on the component | `220` |

The attacker slides itself to `Align Distance` while winding up, then closes to `Strike Reach` during the blow and gives the distance back. `Aim Assist Max Angle` (`50` degrees) is the cone in which the attack looks for a target. Outside the cone you swing at nothing, which is the intended answer, not a bug.

---

## Combat mode is the combat overlay

There is no separate combat flag. The `Combat` overlay is the combat mode, and it is the same overlay scale that arbitrates a drawn pistol or a throwable. See [Weapon overlays and animation layers](../player/weapon_overlays_and_layers.md).

The player enters it when an AI inside `Combat Threat Range` (`500` on `BP_PlayerCharacter`) is targeting the player. An enemy that has not noticed you does not put you in combat, which is what keeps stealth intact. You leave it by killing that enemy, by getting out of range, or by drawing a weapon or a throwable.

The combat camera follows the same overlay, so anything that ends the fight also gives the camera back. Its framing is expressed as a delta from your normal framing, never as an absolute. See [Change the combat camera and the impact feeling](combat_camera_and_impact_feel.md).

---

## Same component, different numbers

| Field | Player | Human NPC | Infected |
|---|---|---|---|
| `Attack Montages` | `AM_Combat_Attack_01`, `AM_Combat_Attack_02` | the same two | `AM_Combat_Attack_01` only |
| `Strike Damage` | `10` | `10` | `12` |
| `Recovery Delay` | `0` | `0.8` | `0.5` |
| `Use Attack Hold` | off | on, `1.2` seconds | off |
| `Dodge Montages` | `AM_Combat_Dodge_01` | `AM_Combat_Dodge_01` | empty |
| `Dodge Chance` | `0` | `0.35` | `0` |
| `Dodge I Frame Time` | `1.5` | `1` | `1` |
| `Stealth Finishers` | one pair | empty | empty |
| `Can Be Finished` | off | on | on |
| `Use Hit Stop` | on | on | off |

Two of these rows are the whole extension story. The infected does not dodge because its `Dodge Montages` is empty, not because a graph checks what class it is: empty data turns the behaviour off. And the player and the NPC share the same two attack montages, so a montage you add to `Attack Montages` on both sides is a new punch for everyone at once.

The player dodges on a key, `IA_Dodge`. An AI dodges on a roll of `Dodge Chance` when it sees an attack coming at it from within reach. Same component, two ways in.

---

## What melee does not do

Said plainly so you do not go looking for it:

- **A melee AI does not circle around you or back off while it waits its turn.** It closes, attacks, and holds its spacing. Turn taking is real, the choreography around it is not.
- **There is no front finisher.** The finisher is the stealth one, from behind. `AM_Combat_Finisher_01` exists but ships without a matching victim animation.
- **Every shipped attack is bare handed.** There is no melee weapon in the template.
- **Only the player has a stealth finisher configured.** `Stealth Finishers` is empty on both AI, and filling it in is all it takes to change that.

Turn on `Log Combat State` on a component and its state is printed as it changes, which is the fastest way to see which of the three refusals is stopping an attack.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
