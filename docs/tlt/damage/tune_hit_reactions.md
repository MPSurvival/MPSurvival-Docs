# Change how hard a hit throws a body

Hit reactions in the template are physical. When a character is hit, the body simulates for half a second, takes an impulse at the point of contact, then blends back to the animation it was playing. There is no hit animation to swap, and no montage to author.

Two things decide what a hit looks like, and you can reach both from a `Details` panel:

- **how hard the body is pushed**, which comes from a hit profile Data Asset;
- **how long and how loose the body stays**, which comes from fields on the `BP_HitReactionManager` component of the character being hit.

---

## The two profiles that ship

They live in `Content/TheLastTemplate/Blueprints/DataAssets/Combat/`:

| Asset | Used for |
|---|---|
| `DA_Hit_Punch` | melee strikes |
| `DA_Hit_Bullet` | bullets |

Both are built on the `BP_HitProfileDataAsset` class, in the same folder.

You never pick a profile per attack. A melee strike uses the victim's `Melee Hit Profile`, a bullet uses the victim's `Bullet Hit Profile`. Both fields sit on `BP_HitReactionManager`, and both are already filled in on the player, the NPC, the zombie and `BP_Dummy`.

So there are two ways to change a reaction:

- edit `DA_Hit_Punch` and every punch in the game gets heavier;
- point one character's `Melee Hit Profile` at a different asset and only that character reacts differently.

There is one melee profile and one bullet profile **per character**, not one per weapon. A shotgun and a pistol push the same body the same way today.

---

## How hard the push is

Three fields on the profile make the impulse, and they are multiplied together:

`Effective Mass` × `Reference Speed` × `Transfer Coefficient`

| Field | What it does | Shipped |
|---|---|---|
| `Effective Mass` | the mass of the thing that hits you, in kilograms | `10` on punch, `0.00745` on bullet |
| `Reference Speed` | the speed of the thing that hits you, in centimetres per second | `914` on punch, `36000` on bullet |
| `Transfer Coefficient` | the share of that momentum that ends up in the body, 0 to 1 | `0.85` on punch, `0.4` on bullet |

That is about 7770 for a punch against 107 for a bullet, so the punch pushes roughly seventy times harder. This is on purpose, not an oversight. A bullet really does carry far less momentum than a fist, and a body thrown backwards by a pistol round is the one thing that instantly reads as fake.

The impulse is applied at the point of contact, never at the centre of the body, so what you actually see is rotation rather than a shove. Small changes to `Transfer Coefficient` show up more than you expect.

If you want bullets to move a body more, raise `Transfer Coefficient` first. Leave `Effective Mass` and `Reference Speed` where they are unless you are describing a real object: they are the two fields that keep the profiles comparable to each other.

---

## Make a new profile

Say you want buckshot to hit harder than a pistol round.

1. In `Content/TheLastTemplate/Blueprints/DataAssets/Combat/`, right click, then **Miscellaneous**, then **Data Asset**. Pick `BP_HitProfileDataAsset` in the class list.
2. Name it `DA_Hit_Buckshot`.
3. Fill `Effective Mass`, `Reference Speed` and `Transfer Coefficient`.
4. Open the character you want it to apply to, select its `BP_HitReactionManager` component, and set `Bullet Hit Profile` to `DA_Hit_Buckshot`.
5. Save.

Step 4 is the part people forget. The profile does nothing until a character points at it.

---

## How long the reaction lasts

These five fields are on the `BP_HitReactionManager` component, not on the profile. Select the component inside the character Blueprint and they are in the `Details` panel. All four characters that ship carry the same values, listed here.

| Field | What it does | Shipped |
|---|---|---|
| `Hit Recover Time` | the length of the whole reaction, in seconds. The body starts fully physical and blends back to the animation over this time. | `0.5` |
| `Hit Body Softness` | how much of its normal stiffness the whole body keeps at the bottom of the hit. `0.12` means it goes almost limp. `1` means no softening at all. | `0.12` |
| `Hit Reflex Delay` | how long the body stays that soft before it starts tightening back up, in seconds. | `0.1` |
| `Hit Refresh Factor` | what a second hit does to the clock of a reaction already running. | `0.5` |
| `Stack Impulse Falloff` | how much each extra hit inside the same reaction is weakened. | `0` |

`Hit Recover Time` is the field you will touch most. It is the only one that changes how long you look at the reaction. `Hit Body Softness` changes how the reaction reads: low values look like a body that gives way, values near `1` look like someone bracing.

Because these live on the component of each character, they are per character. Giving a zombie a longer `Hit Recover Time` than an NPC is one number on one component.

Keep `Hit Reflex Delay` well under `Hit Recover Time`. They ship at `0.1` and `0.5`, so the body stays soft for a fifth of the reaction and spends the rest tightening back up. Set them close together and there is no room left for that.

---

## Hits that land close together

A hit that arrives while a reaction is still running does not restart it.

- The clock is pulled back by `Hit Refresh Factor`: at `0.5` the reaction rewinds halfway, so it lasts longer without snapping back to the start. At `1` it would not extend at all. At `0` it would restart from scratch.
- The new impulse is added on top, weakened by `Stack Impulse Falloff` once for each hit already stacked. It ships at `0`, so every hit of a burst pushes at full strength. Raise it if a rapid fire weapon sends a body too far.

---

## The rest of the profile

`BP_HitProfileDataAsset` has seven fields. Three of them make the impulse. Here are the other four.

| Field | Shipped |
|---|---|
| `Reaction Scalar` | `1` on punch, `2` on bullet |
| `Balance Threshold` | `1500` on both |
| `Responses` | empty on both |
| `Flinch Offsets` | empty on both |

`Responses` holds rows of `S_HitRegionResponse`, one per body region, each with its own `Soft Floor`, `Release Time`, `Reflex Delay` and `Recover Time`. `Flinch Offsets` holds rows of `S_HitBoneOffset`, each with a `Bone`, a rotation `Offset` and a `Weight`.

Both arrays are empty on the two shipped profiles, so there is no example in the project to copy from, and no body region has timing of its own: a hit to the head and a hit to a leg use the same five numbers off the component.

The enum `E_HitImpactType` lists seven names, `Punch`, `Kick`, `Blunt`, `Bite`, `Bullet`, `Buckshot` and `Blast`, but it is not what selects a profile. The two profiles cover the two things that currently produce a reaction: a melee strike and a bullet.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
