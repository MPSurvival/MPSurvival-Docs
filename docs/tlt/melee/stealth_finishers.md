# Add a stealth finisher

A stealth finisher is one row in an array. The row pairs the animation the attacker plays with the animation the victim plays, plus an offset that lines the two bodies up. Fill a row, place one notify, and you have a new takedown.

---

## What ships

One pair, on the player.

| Piece | Asset |
|---|---|
| Attacker montage | `AM_Combat_Stealth_01` |
| Victim montage | `AM_Combat_Stealth_Victim_01` |
| Key | `IA_Finisher`, `F` by default |

Both montages are in `Content/TheLastTemplate/Animations/Montages/Characters/Combat/`.

The list of finishers belongs to the attacker. It is a field called `Stealth Finishers` on the `BP_CombatComponent` of `Content/TheLastTemplate/Blueprints/PlayerCharacter/BP_PlayerCharacter`.

`BP_NPCCharacter` and `BP_ZombieCharacter` carry the same component, and their `Stealth Finishers` array is empty. No AI in the template performs a finisher, on you or on each other. They are victims only.

---

## Change the pair, or add another

1. Open `BP_PlayerCharacter`.
2. Select `BP_CombatComponent` in the Components panel.
3. In the Details panel, find `Stealth Finishers`. It has one element.
4. Set `Attacker Montage` and `Victim Montage` to your own pair, or click the plus to add a second element and fill that.
5. Leave `Victim Offset` at zero for now.
6. Compile and save.

Then place the kill notify, which is the next section, and play.

The array ships with a single element, so there is nothing for the template to choose between. If you add a second element, play it and confirm which one comes out before you build a lot of content on top of it.

---

## The row

`S_StealthFinisher`, in `Content/TheLastTemplate/Blueprints/Structures/`, has three fields and nothing else. A finisher is two animations and a position.

| Field | What it does | Shipped |
|---|---|---|
| `Attacker Montage` | The montage played on you | `AM_Combat_Stealth_01` |
| `Victim Montage` | The montage played on the victim | `AM_Combat_Stealth_Victim_01` |
| `Victim Offset` | Where the victim is put before the two montages play | `(0, 0, 0)` |

---

## Line the two bodies up

The shipped pair needs no correction, which is why `Victim Offset` is `(0, 0, 0)`.

A pair you animate yourself usually does need one. Do not guess the numbers. Play the finisher once, watch where the victim's feet land against yours, then type the correction in centimetres: X forward, Y right, Z up. Move by five or ten at a time and play again. Two or three passes is normal.

If your two animations were exported from the same scene, with both characters at the same origin, zero is the right answer and you should not touch this field at all.

---

## Choose the frame where the victim dies

The kill is an Anim Notify called `AN_FinisherKill`, in `Content/TheLastTemplate/Animations/AnimNotifies/Character/`.

It sits on the **victim** montage, not the attacker one. Open `AM_Combat_Stealth_Victim_01` and you will see it on the notify track, next to an `AN_PlaySound`. Look there first, because that is not where most people go hunting for it.

For a new pair:

1. Open your victim montage.
2. Right click the notify track at the frame where the kill lands and add `AN_FinisherKill`.
3. Select the notify and set `Lethal Damage` in the Details panel.

`Lethal Damage` is the damage dealt at that frame. Put it above the victim's maximum health, with room to spare, so the finisher still kills after you raise enemy health later.

!!! warning "No notify, no kill"
    Nothing else in the pair deals damage. If the victim montage has no `AN_FinisherKill`, both animations play all the way through and the victim stands back up alive.

---

## Decide who can be finished

`Can Be Finished` is a tick box on the victim's own `BP_CombatComponent`. A character with it off is never offered as a target.

| Character | `Can Be Finished` |
|---|---|
| `BP_PlayerCharacter` | off |
| `BP_NPCCharacter` | on |
| `BP_ZombieCharacter` | on |

Tick it on any character you want to be a valid victim, and untick it to make one immune. The player ships with it off.

---

## Set the reach and the angle

Two more fields on the attacker's `BP_CombatComponent`. Both ship at the same value on the player, the NPC and the zombie.

| Field | Shipped | What it does |
|---|---|---|
| `Stealth Reach` | `150` | How close you have to be, in centimetres |
| `Stealth Behind Dot` | `0.5` | How well lined up behind the victim you have to be |

`Stealth Behind Dot` is a dot product, so it runs from `1` to `0`. At `0.5` you have to come in roughly from behind, inside a cone of about sixty degrees. Push it toward `1` and only a dead centre approach works, which feels strict but reads as deliberate. Drop it toward `0` and you can take someone from the side, which is generous and starts to look wrong on camera.

`Stealth Reach` at `150` is a bit more than one character width. Raising it much past that lets the finisher start with a visible gap between the two bodies, and no `Victim Offset` will hide that.

---

## The prompt over the victim

`BP_NPCCharacter` carries a Widget Component called `Finish Widget`. That is the prompt that appears above a victim you are in position to take down. Its `Widget Class` decides what is drawn.

The widget that ships is `BP_FinishWidget`, in `Content/TheLastTemplate/Blueprints/Widgets/Combat/`. It holds a `Key Image` for the key icon and a `Finish Animation` for the fade.

The icon on that image follows the key mapping, so if you move the finisher off `F` the prompt updates by itself. See [Change the controls](../start/change_the_controls.md) for the mapping, and [Key icons and letting players rebind keys](../ui/key_prompts_and_rebinding.md) for how the icons are drawn.

---

## Before you call it done

- The prompt appears when you walk up behind an enemy, and disappears when you step to the side.
- Pressing the key starts both montages at the same moment.
- Feet and hands stay together for the whole animation. If they drift apart, that is `Victim Offset`.
- The victim dies on the frame you chose, not at the end of the montage.
- The victim stays down.

---

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
