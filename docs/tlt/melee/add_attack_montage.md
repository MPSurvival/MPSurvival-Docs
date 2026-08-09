# Add a new melee attack

You end up with your own punch or kick: an animation a character throws the same way it throws the two attacks that ship, with a strike window you place yourself on the timeline.

Two attacks ship, both in `Content/TheLastTemplate/Animations/Montages/Characters/Combat/`:

| Montage | Length | Listed on |
|---|---|---|
| `AM_Combat_Attack_01` | 0.9 s | player, human NPC, infected |
| `AM_Combat_Attack_02` | 1.7 s | player, human NPC |

The work is the same whichever character gets the new one. If you have not read [How melee combat works](melee_how_it_works.md), read it first: it says what refuses an attack and what the strike sweep is allowed to hit.

---

## Before you start

You need an animation sequence on `S_HumanoidCharacter`, the skeleton at `Content/TheLastTemplate/Meshes/Characters/S_HumanoidCharacter`. Anything retargeted onto it works. See [Use your own animations](../player/use_your_own_animations.md) if the animation is not on that skeleton yet.

The sequences behind the shipped attacks are in `Content/TheLastTemplate/Animations/AnimSequences/Characters/Combat/`. Open `AS_Combat_Attack_01` beside yours if you want a reference for timing.

---

## 1. Make the montage

1. Right click your animation sequence, then **Create**, then **Create AnimMontage**.
2. Name it in the pattern of the others, `AM_Combat_Attack_03`, and put it in `Content/TheLastTemplate/Animations/Montages/Characters/Combat/`.
3. Open it and set the slot to `Combat`. `S_HumanoidCharacter` declares `Combat` next to `DefaultSlot` and `UpperBody`, and both shipped attacks play on `Combat`.
4. Leave the blends alone unless you have a reason. Both shipped attacks blend in over `0.1` and out over `0.25`.

The shipped attack sequences have **Enable Root Motion** on, and `ABP_HumanoidCharacter` takes root motion from montages only. So whatever your animation does to the root, the character does in the level. A lunge in the animation is a lunge in the game.

---

## 2. Place the strike window

Nothing is hit because a hand moves. Damage is only possible inside the window of an `ANS_MeleeStrike` notify state, and where you put that window is the timing of the blow.

1. In the montage editor, add a notify track if you want the strike on its own line.
2. Right click the track at the frame where the fist starts to travel, then **Add Notify State**, then `ANS_MeleeStrike`.
3. Drag the right edge of the window to the frame where the fist has gone past.
4. Select the notify state and fill its three fields in the Details panel.

| Field on the notify | What it does | On a fresh notify |
|---|---|---|
| `Hand Bone` | the bone the sweep follows | `Hand_R` |
| `Hit Radius` | radius of the sweep, in centimetres | `45` |
| `Target Region` | which part of the victim this animation aims at | `Head` |

`Target Region` is the one to think about. The animation declares what it aims at, and that is what the physical reaction on the victim starts from. A hook to the jaw is `Head`, a body blow is `UpperTorso`, a low kick is `LegL` or `LegR`. The whole list is `E_HitRegion`: `Head`, `Neck`, `UpperTorso`, `LowerTorso`, `ArmL`, `ArmR`, `LegL`, `LegR`.

Be generous with the window. One swing lands one hit, so a window covering the full travel of the arm does not hit twice, it only stops you missing on a frame.

!!! warning
    A montage with no `ANS_MeleeStrike` on it is not an attack. It plays, the character swings, nothing is ever hit, and there is no error anywhere. `AM_Combat_Dodge_Attack_01` ships in that same folder in exactly that state: an animation, not a working attack.

---

## 3. Give it to a character

1. Open `BP_PlayerCharacter`, at `Content/TheLastTemplate/Blueprints/PlayerCharacter/BP_PlayerCharacter`.
2. Select `BP_CombatComponent` in the **Components** list.
3. In the Details panel, find `Attack Montages`, add an element, and pick your montage.
4. Compile and save.

Repeat on the AI that should throw it too:

- `Content/TheLastTemplate/Blueprints/AI/NPC/BP_NPCCharacter`
- `Content/TheLastTemplate/Blueprints/AI/Zombie/BP_ZombieCharacter`

The array is per character. A montage added to the player only stays the player's, which is how the infected ends up with one attack while the humans have two.

---

## The numbers around the attack

Same component, same Details panel, further down:

| Field | What it does | Player | Human NPC | Infected |
|---|---|---|---|---|
| `Strike Damage` | damage a landed strike deals | `10` | `10` | `12` |
| `Recovery Delay` | pause at the end of an attack before the character can act again | `0` | `0.8` | `0.5` |
| `Use Attack Hold` | keep a minimum delay between two attacks | off | on | off |
| `Attack Hold Time` | that delay, in seconds | `1.2` | `1.2` | `1.2` |

Two things follow from that table.

**Damage is per character, not per montage.** Every attack the infected throws does `12`, whichever animation was drawn. There is no field that makes one montage hit harder than another.

**The pace of a fighter is `Recovery Delay` plus the length of the montage.** A 1.7 second animation is already a slower attack than a 0.9 second one, with no field involved, so a heavy attack is something you animate long rather than something you configure. `Use Attack Hold` is the third lever and it ships on for the human NPC only: it is what stops a human chaining blows on you while the infected keeps coming.

---

## How often an AI throws it

An AI never names a montage. It asks the component and the component draws from `Attack Montages`, so your new attack joins the pool the moment you add it, with no behavior tree edit.

How often it asks, and how many blows it strings together, live on the AI Data Asset instead: `Melee Cooldown` (`1.25`), `Combo Min` (`2`), `Combo Max` (`3`), `Combo Delay` (`0.35`) and `First Attack Delay`, which is `2` on the human presets and `0.3` on `DA_Zombie_Roaming`. See [Tune a melee AI](..

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
