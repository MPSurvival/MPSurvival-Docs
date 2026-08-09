# Add or change the dodge

The dodge is one array on the combat component. Fill `Dodge Montages` on a character and that character can dodge. Leave it empty and it never will, and nothing else has to change.

Everything on this page is in one Details panel. Open the character Blueprint, select `BP_CombatComponent` in the Components list, and look at the `Dodge` category. The same three fields exist on the player, on the human NPC and on the infected, because it is the same component on all three. See [How melee combat works](melee_how_it_works.md).

The key is `IA_Dodge`, on **Space** by default. See [Change the controls](../start/change_the_controls.md).

---

## The three fields

| Field | What it does | Note |
|---|---|---|
| `Dodge Montages` | The dodge animations. An empty array means this character cannot dodge. | It is an array, so it can hold more than one |
| `Dodge I Frame Time` | How long, in seconds, the dodge protects you, counted from the moment it starts | |
| `Dodge Chance` | Chance from 0 to 1 that an AI dodges a strike it sees coming | Read by AI only |

What each character ships with:

| Character | `Dodge Montages` | `Dodge I Frame Time` | `Dodge Chance` |
|---|---|---|---|
| `BP_PlayerCharacter` | `AM_Combat_Dodge_01` | `1.5` | `0` |
| `BP_NPCCharacter` | `AM_Combat_Dodge_01` | `1` | `0.35` |
| `BP_ZombieCharacter` | empty | `1` | `0` |

`Dodge Chance` is `0` on the player because the player dodges on a key press, never on a roll. Raising it on `BP_PlayerCharacter` does nothing.

---

## The movement comes from the animation

There is no dash code, no launch, no curve. The character moves because the dodge montage carries root motion. Distance, direction and speed of the dodge all live in the animation you pick, so a dodge that is too short or too fast is fixed in the animation, not in a field.

!!! warning
    An animation with **Enable Root Motion** off plays in place. You get the pose and the protection window, and no displacement at all. Nothing logs an error, so this is easy to spend an hour on.

---

## Add your own dodge animation

1. Have your animation as an Anim Sequence on `S_HumanoidCharacter`. The combat sequences that ship are in `Content/TheLastTemplate/Animations/AnimSequences/Characters/Combat/`.
2. Open the sequence and tick **Enable Root Motion**.
3. Right click it, then **Create**, then **Create AnimMontage**. Save it next to the others, in `Content/TheLastTemplate/Animations/Montages/Characters/Combat/`.
4. Open `AM_Combat_Dodge_01` and give your montage the same slot. A montage in the wrong slot plays on the wrong half of the body.
5. Open the character Blueprint and select `BP_CombatComponent`. Under `Dodge Montages`, add an element and pick your montage.
6. Compile, save, and test in the level.

`AM_Combat_Dodge_Attack_01` also ships in that folder. It is not assigned to the player, to the human NPC or to the infected, so it is free for you to use or replace.

---

## When the player is allowed to dodge

Three things have to be true. All three are honest limits, not bugs.

- **Empty hands.** A gun or a throwable in hand puts the character on a different overlay and the dodge is refused. This check sits on the player, not on the shared component, so it never restricts AI.
- **An enemy is on you.** The dodge needs combat mode, which means an AI that has taken the player as its target, inside `Combat Threat Range` (`500` on `BP_PlayerCharacter`). It is the same signal that pulls the camera in, so if the combat camera is active you can dodge. See [Change the combat camera and the impact feeling](combat_camera_and_impact_feel.md).
- **Not busy.** You cannot dodge out of your own punch, out of another dodge, out of a stagger, or out of a finisher.

Waiting for your attack turn does **not** block a dodge. That was left open deliberately: the attack pacing should delay your next punch, not stop you from getting out of the way.

---

## The window where nothing can touch you

`Dodge I Frame Time` is a plain duration in seconds, started the instant the dodge begins. It is not tied to the length of the montage. A longer animation does not buy you a longer window, and a short one does not cut it short. Tune the two separately.

The protection is targeted. It covers you against the enemy whose strike you dodged, and against that enemy only. A second attacker standing beside them still lands their hit. There is one exception: if no attacker had been identified when you dodged, the window protects you from everyone for its duration. Threat detection runs a few times per second, so this happens on a dodge thrown before anything has been seen.

### Keep it under the attacker's `Attack Hold Time`

`Attack Hold Time` is in the same panel, on the attacking character, and it is the gap that character waits before it may strike again. `BP_NPCCharacter` ships `1.2` with `Use Attack Hold` on. If your `Dodge I Frame Time` is longer than that gap, a single dodge also carries you through the enemy's next strike.

The player ships at `1.5`, which is above `1.2`, so a player dodge currently covers the follow up too. Set it under `1.2` if you want one dodge to buy exactly one strike. Go the other way, past `2`, and a well timed dodge makes the player untouchable by that enemy for two swings.

`Use Attack Hold` is off on `BP_PlayerCharacter` and on `BP_ZombieCharacter`. The infected has no hold to compare against, so against infected only the duration matters.

---

## Give a character no dodge at all

Leave `Dodge Montages` empty. That is exactly how the infected ship: `BP_ZombieCharacter` has an empty array, so it never dodges, and no switch had to be turned off. There is no separate checkbox, and there is no branch per class. The data turns the feature off.

The same applies to any character you build. A new enemy type that should never dodge is a component with one empty array.

---

## Let an AI dodge on its own

An AI needs no input. When it sees its current opponent start a strike, and that opponent is within `Max Attack Range` (`220` on all three characters), it rolls once against `Dodge Chance`. One roll per incoming strike, not one per frame, so raising the value makes dodges more likely and never makes them spammy.

- `0` never dodges.
- `0.35`, the human NPC value, slips roughly one strike in three.
- `1` dodges every strike it sees coming, which is very hard to fight.

`Dodge Chance` and a filled `Dodge Montages` are both required. Either one missing and the AI stands and takes it.

Two limits to know. The AI only reacts to the opponent it is already fighting, so a strike from a third party is not dodged. And nobody backs off or circles while waiting for their turn, so a dodging NPC steps aside and comes straight back in. For the rest of the melee AI numbers, see [Tune melee AI](../ai/tune_melee_ai.md).

---

## Check it works

1. Place a `BP_NPCCharacter` that is hostile to the player and walk up to it with empty hands. The camera pulling in tells you combat mode is on.
2. Press **Space**. The character must travel. If it plays the animation and stays put, root motion is off on the sequence.
3. Let the NPC swing and press **Space** as the swing starts. The punch should pass through you.
4. Set `Dodge Chance` to `1` on `BP_NPCCharacter`, then punch at it. It should slip every strike.
5. Put `Dodge I Frame Time` at `0.1` on the player and try again. The dodge still moves you, but a strike now connects, which confirms the field is the only thing controlling the window.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
