# Tune how the AI fights up close

Close range AI is tuned in two places, and neither of them is a graph.

- **The AI Data Asset** decides when an enemy comes at you, how it closes the gap, and how near it gets before it swings. The presets live in `Content/TheLastTemplate/Blueprints/DataAssets/AI/Childs/`.
- **The `BP_CombatComponent` on the character** decides what the swing does: which animation plays, how much it hurts, how far it reaches, how long the fighter is busy afterwards. You edit it on `BP_NPCCharacter` or `BP_ZombieCharacter`, not on the component asset.

Two melee fighters ship: the NPC, in `Content/TheLastTemplate/Blueprints/AI/NPC/BP_NPCCharacter`, and the zombie, in `Content/TheLastTemplate/Blueprints/AI/Zombie/BP_ZombieCharacter`. They are sibling classes, so a change on one does not reach the other.

For how the pieces fit together, see [How AI combat is put together](ai_combat_overview.md).

---

## Closing in

These fields are on the AI Data Asset. Open a preset such as `DA_NPC_Melee_Roaming` or `DA_Zombie_Roaming` and they are all in one list.

| Field | What it does | Shipped value |
|---|---|---|
| `Melee Enter Range` | An NPC switches to fighting up close when its target is nearer than this, in centimetres | `200` on every preset |
| `Melee Exit Range` | It goes back to shooting when the target gets further away than this | `280` on every preset |
| `Melee Run Distance` | Further than this, the NPC runs to close the gap instead of walking | `450` on every preset |
| `Strike Range` | How close the NPC gets before it stops and strikes | `130`, and `150` on `DA_NPC_Path` |
| `Strike Stop Margin` | The slack allowed around `Strike Range`, so the NPC is not sliding back and forth | `15`, and `20` on `DA_NPC_Path` |
| `First Attack Delay` | How long a zombie waits after reaching you before its first swing, in seconds | `0.3` on `DA_Zombie_Roaming` |

`Melee Enter Range` and `Melee Exit Range` are deliberately different numbers. The gap between them is what stops an NPC flipping between the gun and its fists while you walk in and out of range. Keep a gap when you change them.

An NPC that only ever fights with its hands is made by clearing `Default Weapon Class` on its Data Asset. That is the whole difference between `DA_NPC_Roaming` and `DA_NPC_Melee_Roaming`. There is no melee mode flag.

Two of these do not apply to the zombie. `BT_Zombie` uses its own chase task, which does not read `Strike Range` or `Strike Stop Margin`, so the `100` and `20` sitting on `DA_Zombie_Roaming` change nothing. `First Attack Delay` is the opposite case: only the zombie reads it, and the `2` on the NPC presets is never used.

---

## The strike

Open `BP_NPCCharacter`, select `BP_CombatComponent` in the Components list, and the fields below are in the Details panel.

| Field | What it does | On the NPC | On the zombie |
|---|---|---|---|
| `Attack Montages` | The attack animations. One is picked per swing | `AM_Combat_Attack_01`, `AM_Combat_Attack_02` | `AM_Combat_Attack_01` |
| `Strike Damage` | Health removed by one landed strike | `10` | `12` |
| `Strike Reach` | How far past the hand the strike looks for a target, in centimetres | `80` | `80` |
| `Default Hit Radius` | How thick that search is, in centimetres | `60` | `60` |
| `Use Both Hands` | Test both hands instead of only `Default Hand Bone` | `true` | `true` |
| `Hand Bone Left` / `Hand Bone Right` | The bones the strike is measured from | `Hand_L` / `Hand_R` | same |
| `Recovery Delay` | How long the fighter is busy after a strike, in seconds | `0.8` | `0.5` |

`Recovery Delay` is the single most useful number on this page. It is the pause between one attack ending and the next one being allowed, so it sets the whole rhythm of a fight. `0.8` on the NPC reads as a trained fighter picking its moment. `0.5` on the zombie reads as something that does not think about it.

The attack montages live in `Content/TheLastTemplate/Animations/Montages/Characters/Combat/`. To put your own animation in the list, see [Add an attack montage](../melee/add_attack_montage.md), which covers the notifies an attack montage needs.

---

## Spacing and timing

These are on the same component, and both AI characters leave them at the component's own values. What you read here is what runs.

| Field | What it does | Value |
|---|---|---|
| `Max Attack Range` | The furthest a target can be for an attack to start, in centimetres | `220` |
| `Min Combat Distance` | How close a fighter is allowed to stand to its target | `100` |
| `Align Distance` | Inside this distance the fighter turns to face its target before striking | `115` |
| `Align Time` | How long that turn takes, in seconds | `0.3` |
| `Aim Assist Max Angle` | How far off centre a target can be and still be the one you swing at, in degrees | `50` |
| `Attack Hold Time` | How long the combat pose is held after an attack, in seconds | `1.2` |
| `Use Attack Hold` | Turns that hold off | `true` |

`Min Combat Distance` is what keeps two fighters from standing inside each other. Raise it and a crowd spreads out, lower it and they crush in. `Max Attack Range` has to stay comfortably above it or an enemy will walk to a spot it cannot attack from.

Hit stop, camera shake, knockback and the impact sound are on this component too, and they are shared with the player. They are covered in [Combat camera and impact feel](../melee/combat_camera_and_impact_feel.md).

---

## Letting an enemy dodge

Dodging is off on the component by default and turned on per character.

| Field | What it does | On the NPC | On the zombie |
|---|---|---|---|
| `Dodge Montages` | The dodge animations | `AM_Combat_Dodge_01` | empty |
| `Dodge Chance` | Chance of dodging an incoming attack, from `0` to `1` | `0.35` | `0` |
| `Dodge IFrame Time` | How long a dodging fighter cannot be hit, in seconds | `1` | `1` |

So one NPC attack in three is answered with a sidestep, and a zombie never dodges. Raising `Dodge Chance` on the zombie will not do anything on its own, because its `Dodge Montages` list is empty. Fill the list first.

The player's dodge is a separate thing with its own key. See [The player dodge](../melee/player_dodge.md).

---

## Letting an enemy be finished

| Field | Where | What it does | Shipped value |
|---|---|---|---|
| `Can Be Finished` | `BP_CombatComponent` | Whether a stealth finisher can be started on this character | `true` on both |
| `Finish Automatically` | the character | Whether the finisher runs on its own once you are in position | `true` on both |
| `Finish Direct` | the character | An immediate finish instead of the prompted one | `false` on both |

Set `Can Be Finished` to `false` on a character you want to be immune, for example a boss you do not want removed with one button.

The distance and the angle a finisher is allowed from are on the attacker, not the victim, so they are the same for every enemy. [Stealth finishers](../melee/stealth_finishers.md) covers them.

---

## Two enemies from the same parts

A heavy enemy, made from a copy of `BP_NPCCharacter`:

1. Duplicate `BP_NPCCharacter` and give the copy its own Data Asset.
2. On its `BP_CombatComponent`, raise `Strike Damage` to `20` and `Recovery Delay` to `1.2`.
3. Set `Dodge Chance` to `0`. Heavy things do not sidestep.
4. On its Data Asset, raise `Melee Run Distance` so it walks most of the way.

A fast enemy, from the same copy:

1. Lower `Recovery Delay` to `0.4` and `Strike Damage` to `6`.
2. Raise `Dodge Chance` to `0.6` and keep at least two entries in `Dodge Montages`.
3. Lower `Melee Run Distance` so it starts running almost at once.
4. Lower `Strike Range` a little on its Data Asset so it presses right up against you.

Both changes are Details panel values on a duplicated class. The full recipe for a new enemy class is in [Add a new enemy type](add_new_ai_type.md).

---

## Fields that do nothing today

The AI Data Asset carries a set of close range fields that nothing in the project reads. They hold values, they can be edited, and no behaviour changes. They are listed here so you do not spend an evening tuning them:

- rhythm: `Melee Cooldown`, `Combo Min`, `Combo Max`, `Combo Delay`
- facing: `Face Target On Attack`, `Face Turn Interp Speed`
- backing off: `Run Away Speed Threshold`, `Run Away Dot Threshold`, `Retreat Distance`, `Retreat Repath Interval`
- circling: `Ring Repick Interval`, `Ring Repick Jitter`, `Ring Strafe Distance`, `Ring Snap Distance`
- finishers: `Finisher Range`, `Finisher Facing Dot`
- difficulty: `Melee Defense Level`
- on the zombie asset: `Attack Damage`, `Strike Bone`

`BT_NPC` does have tasks that circle and retreat, but they read none of the fields above, so there is no number to turn. The rhythm of a fight comes from `Recovery Delay` on the component and from the length of your attack montages, not from `Melee Cooldown`.

!!! warning "The zombie's damage is not on its Data Asset"
    `DA_Zombie_Roaming` shows an `Attack Damage` of `12` and nothing reads it. The number that applies is `Strike Damage` on the zombie's `BP_CombatComponent`, which is also `12`. Change the Data Asset and zombies keep hitting for the same amount. Change the component.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
