# Change the combat camera and the impact feeling

Two things change how a fight feels: where the camera sits while you are fighting, and what happens the frame a punch lands. Both are Details panel fields. You never open a graph for either.

They live in two different places:

- **The camera** is on the player, in `BP_PlayerCharacter`, under the `Settings | Combat` category.
- **The impact** is on the combat component, `BP_CombatComponent`, which every fighter carries.

---

## The combat camera

While the combat overlay is on, the player pushes a camera override: the camera moves by `Combat Camera Offset` and the boom shortens by `Combat Boom Delta`. When combat ends, the override is dropped and the camera goes back to normal on its own.

Open `Content/TheLastTemplate/Blueprints/PlayerCharacter/BP_PlayerCharacter`, select the Blueprint defaults, and look under `Settings | Combat`.

| Field | What it does | Shipped |
|---|---|---|
| `Use Combat Camera` | Turn the whole combat camera off. The camera then stays where it always is. | `true` |
| `Combat Camera Offset` | Moves the camera while you fight. X is forward, Y is right, Z is up. | `0, 35, -10` |
| `Combat Boom Delta` | Added to the normal boom length. A negative number pulls the camera closer. | `-70` |
| `Align Camera To Target` | Turn the camera toward the enemy you are fighting during a melee exchange. | `true` |
| `Combat Aim Time` | How long, in seconds, one of those turns lasts. | `0.35` |
| `Combat Aim Speed` | How fast the camera turns during that window. Higher is snappier. | `4` |
| `Combat Threat Range` | How close an enemy must be before you are considered in combat, in centimetres. | `500` |

The shipped setup shifts the framing 35 to the right and 10 down, and pulls the camera 70 closer. That is a tighter over the shoulder shot than the normal one.

`Combat Boom Delta` is a difference, not a length. The length it is added to comes from the `boomTPS` spring arm on the player. If your camera is too far away in general, change `boomTPS`, not this field.

!!! warning
    These fields only do anything while the combat overlay is on. If you change them and play in an empty level, nothing moves and it looks broken. Get an enemy to notice you first, then judge.

### Change the framing

1. Open `BP_PlayerCharacter`.
2. In the Details panel, find `Settings | Combat`.
3. Set `Combat Camera Offset` and `Combat Boom Delta`.
4. Compile and save.
5. Play, walk up to an enemy until the combat stance starts, and look.

For a camera over the left shoulder, put a negative number in Y. For a shot that looks up at the fight, raise Z.

---

## When combat mode starts and stops

Five times a second the player looks for a threat. A character counts as a threat when it is alive, it is closer than `Combat Threat Range`, and it has picked you as its target. If at least one exists, the combat overlay turns on and the camera override goes with it.

So `Combat Threat Range` is not only a camera field. It also decides when your character raises its fists. Raise it and you enter the combat stance from further away. Lower it and enemies get close before anything changes.

The overlay itself is covered in [How melee combat works](melee_how_it_works.md).

---

## The impact

Every fighter has a `BP_CombatComponent`. Select it in the Components list of `BP_PlayerCharacter`, `BP_NPCCharacter` or `BP_ZombieCharacter` and the fields are in its Details panel.

| Field | What it does | Shipped |
|---|---|---|
| `Impact Sound` | The sound played where the hit lands. | `SC_Punch` |
| `Use Hit Stop` | Turn the short slow motion on a landed hit on or off. | `true` on the player and the NPC, `false` on the infected |
| `Hit Stop Scale` | Game speed during that slow motion. `1` is normal speed, lower is slower. | `0.35` |
| `Hit Stop Time` | How long it lasts, in seconds. | `0.09` |
| `Hit Shake Scale` | Multiplier on the camera shake played on impact. `0` removes the shake. | `1` |
| `Knockback Speed` | How hard the victim is pushed, in centimetres per second. | `250` |
| `Impulse Along Aim` | Whether that push is sent along your aim direction. | `true` |

Hit stop scales global time, so it slows everything for those few hundredths of a second, not just the two fighters. That is what sells the weight, and it is also why the numbers are small. Anything past about `0.2` seconds starts to feel like a stutter rather than a hit.

The infected ship with `Use Hit Stop` off on purpose. They hit often and in groups, and a freeze on every one of their swings turns the fight into a slideshow.

### Change the impact sound

1. Open the character you want to change, for example `Content/TheLastTemplate/Blueprints/PlayerCharacter/BP_PlayerCharacter`.
2. Select `BP_CombatComponent` in the Components list.
3. Set `Impact Sound` to your own Sound Cue or Sound Wave.
4. Compile and save.

The shipped cue is `Content/TheLastTemplate/Audios/Combat/Punch/SC_Punch`, and the five punch recordings it draws from sit next to it in the same folder. Replacing the recordings inside the cue is often less work than making a new one.

### Change the shake itself

`Hit Shake Scale` only scales the shake. The motion is in the camera shakes at `Content/TheLastTemplate/Blueprints/Misc/CameraShakes/`, `CS_HitCamera` and `CS_HitCamera_02`. Open one and change its oscillation if you want a different kind of jolt, a longer settle or a roll instead of a shove.

---

## Two presets to copy

These are starting points, not shipped values. Set them on the combat component of the character you want to change.

| Field | Shipped | Softer, faster fights | Heavier, slower fights |
|---|---|---|---|
| `Hit Stop Scale` | `0.35` | `0.6` | `0.15` |
| `Hit Stop Time` | `0.09` | `0.05` | `0.14` |
| `Hit Shake Scale` | `1` | `0.5` | `1.6` |
| `Knockback Speed` | `250` | `150` | `400` |
| `Combat Boom Delta` | `-70` | `-40` | `-100` |

Change one line at a time and play between each. Impact feel is the one thing you cannot judge from the Details panel.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
