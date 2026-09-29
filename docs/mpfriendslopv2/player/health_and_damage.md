# Health, damage and new vitals

Health and stamina are two rows of one array on `BP_VitalsSystem`. This page shows how to hurt a player, heal one, read the values, and add a third vital of your own.

- The component: `Content/MPFriendslop/Blueprints/ActorComponents/BP_VitalsSystem`
- The vital Data Asset class: `Blueprints/DataAssets/Vitals/BP_VitalData`
- The vital Data Assets: `Blueprints/DataAssets/Vitals/Childs/`
- The list of vital types: `Blueprints/Enumerations/Vitals/E_VitalsType`

Other systems never cast to the character to find the vitals. They call `GetVitalComponent` from `BPI_VitalManagerInterface` on the pawn, and the pawn returns its `BP_VitalsSystem`. Do the same in your own Blueprints.

---

## The component

The fields are on `BP_VitalsSystem`, in `Settings|Vitals`.

| Field | What it does | Shipped on the player |
|---|---|---|
| `Vitals` | One row per vital: its Data Asset, its `Current Amount`, and `Pause Decrementation` to stop it draining | `DA_Health_Vital` at `100`, `DA_Stamina_Vital` at `100` |
| `Invulnerable` | When on, health does not go down: damage and `RemoveVitalAmount` on health do nothing. Healing still works. The component also turns it on at death, so a dead player cannot die twice | off |
| `Damage Vital Type` | The vital that engine damage takes from | `Health` |
| `Impact Flash Material` | The overlay put on every mesh of the owner when it takes damage, seen by every player | `M_ImpactFlash` |
| `Impact Flash Duration` | How long that flash lasts, in seconds | `0.5` |
| `Kill Credit Seconds` | A death counts as a kill for the last player who damaged the owner within this many seconds. `OnKilled` gives that player | `5` |

Only `Health` kills. Any other vital can reach 0 without killing anyone.

The same component makes an enemy damageable: `BP_EnemyBase`, the parent of the Warden, carries one too. For damage on actors that are not the player, see [How weapons work](../weapons/how_weapons_work.md).

---

## Hurt a player

Call Unreal's own `Apply Damage` on the player pawn, **on the server**. That is all. The component listens to the owner's `On Take Any Damage`, removes the damage from `Damage Vital Type`, and plays the impact flash.

Anything that already deals engine damage (a gun, a melee swing, your own trap) hurts players with no extra work.

To kill outright instead, skip the vitals: call `Kill` on the pawn's `BP_DeathComponent`, on the server. `BP_DebugKillVolume` in `Blueprints/Environments/` does exactly that when a pawn walks in. See [Death, revive and the spectator](../death/how_death_works.md).

If you need to take from a vital other than health, call `RemoveVitalAmount` with the vital type and the amount, on the server. The value stops at 0.

---

## Heal a player

Call `AddVitalAmount` with the vital type and the amount, on the server. The value stops at the `Vital Max Amount` of that vital's Data Asset.

To heal from an item, there is nothing to call: an item with `Restored Vital` and `Restore Amount` does it through `BP_ConsumableComponent`. `DA_Item_Medkit` restores `50` health. See [Add an inventory item](../items/add_an_item.md).

---

## Read a value

1. Call `GetVitalComponent` on the pawn (the `BPI_VitalManagerInterface` message).
2. Call `GetVital` on the result with the vital type.
3. Break the returned `S_Vitals` and read `Current Amount`.

For the maximum, read `Vital Max Amount` on the row's `Vital Asset`. `GetIsDead` tells you if the player is dead.

To react to a change instead of polling, bind a dispatcher.

| Dispatcher | When it fires | Fires on |
|---|---|---|
| `OnVitalsChanged` | Any vital changed. Bind this one for a HUD | every player |
| `OnVitalDepleted` | A vital just reached 0, with its type | server |
| `OnDied` | Health reached 0 | server |
| `OnKilled` | Health reached 0. Gives `Victim` and `Killer`, the controller of the last player who damaged it within `Kill Credit Seconds`, or none. The arena counts its kills with it | server |
| `OnDamageTaken` | On `BP_DamageFeedbackComponent`: the watched vital went down, with the amount | owning player |

---

## The damage feedback

`BP_DamageFeedbackComponent` on the pawn gives the hurt player a sound, a camera shake and a red screen edge. It only runs on the hurt player's own machine and sends nothing over the network.

| Field | What it does | Shipped value |
|---|---|---|
| `Watched Vital` | The vital whose drop counts as a hit. Empty turns the component off | `DA_Health_Vital` |
| `Damage Sound` | Played in 2D on a hit | `CUE_Damage_Hurt` |
| `Damage Shake` | Camera shake played on a hit | `CS_DamageHit` |

The fields are in `Settings|Damage`. The red edge is `WBP_DamageVignette`, inside `WBP_Gameplay`, and it listens to `OnDamageTaken`. Bind the same dispatcher for your own hit effect or rumble.

Sprinting drains stamina, not health, so it does not flash the screen.

---

## Add a vital

A hunger, oxygen or sanity bar, for example.

1. Open `E_VitalsType` and add an entry, for example `Hunger`.
2. In `Blueprints/DataAssets/Vitals/Childs/`, duplicate `DA_Stamina_Vital` and name it `DA_Hunger_Vital`.
3. Open it and set the fields below. They come from `BP_VitalData`, the class of every vital Data Asset.
4. Open `BP_FriendslopCharacter`, select `BP_VitalsSystem`, and add a row to `Vitals` with your Data Asset and a starting `Current Amount`.

| Field | What it does | `DA_Health_Vital` |
|---|---|---|
| `Vital Type` | Which `E_VitalsType` entry this asset is | `Health` |
| `Vital Color` | The colour of this vital's glyph and number on the HUD | green |
| `Vital Max Amount` | The ceiling for healing | `100` |
| `Vital Tick Decrementation` | Removed every 0.1 seconds on the server. `1` means 10 per second. `0` means it never drains | `0` |

Your vital now drains, heals, fires the dispatchers and reaches every player. It does not show on the HUD yet: `WBP_Vitals` draws two fixed rows, `HealthRow` and `StaminaRow`, not one row per entry of `Vitals`. Showing a third one is an edit of `WBP_Vitals`: a copy of `StaminaRow` with your own icon, styled in `AttachToPawn` and refreshed in `RefreshValues` like the other two. The steps are on [The HUD, and adding to it](../ui/the_hud.md#show-a-new-vital).

!!! warning
    A pawn that has `BP_VitalsSystem` but does not implement `BPI_VitalManagerInterface` is invisible to the rest of the template: no HUD values, no stamina, no damage feedback, and the Warden will not target it. There is no error. See [How the player character works](how_the_player_works.md#your-own-character-blueprint).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
