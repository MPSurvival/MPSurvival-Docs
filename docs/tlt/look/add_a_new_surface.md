# Add a new surface

Six surfaces ship, plus Default. This page adds a seventh from end to end, so a bullet fired at it throws the right debris, leaves the right hole, and your footsteps sound right on it.

The example here is **Sand**. It touches five assets and two Blueprints, and none of it needs a new system.

If you have not read [How surfaces work](how_surfaces_work.md), read it first. It explains what a surface actually decides.

---

## What ships today

The surfaces are declared once in the project, and one physical material carries each of them.

| Surface | Value in the project | Physical material |
|---|---|---|
| Default | `SurfaceType_Default` | `PM_Default` |
| Metal | `SurfaceType1` | `PM_Metal` |
| Wood | `SurfaceType2` | `PM_Wood` |
| Grass | `SurfaceType3` | `PM_Grass` |
| Blood | `SurfaceType4` | `PM_Blood` |
| Glass | `SurfaceType5` | `PM_Glass` |
| Water | `SurfaceType6` | `PM_Water` |

All seven live in `Content/TheLastTemplate/Materials/PhysicsMaterials/`.

The three asset families that react to a surface do not each cover all seven. Check the list before you pick something to copy:

- Impact effects, in `Content/TheLastTemplate/Niagara/Impacts/`: `NS_Impact_Blood`, `NS_Impact_Concrete`, `NS_Impact_Grass`, `NS_Impact_Metal`, `NS_Impact_Water`, `NS_Impact_Wood`. There is no glass one.
- Decal instances, in `Content/TheLastTemplate/Materials/Effects/Decals/Instances/`: `MI_Decal_Impact_Blood`, `MI_Decal_Impact_Concrete`, `MI_Decal_Impact_Dirt`, `MI_Decal_Impact_Glass`, `MI_Decal_Impact_Metal`, `MI_Decal_Impact_Wood`. There is no grass one and no water one.
- Footstep cues, in `Content/TheLastTemplate/Audios/Footsteps/`: `SC_FS_Concrete`, `SC_FS_Grass`, `SC_FS_Ground`, `SC_FS_Metal`, `SC_FS_Water`, `SC_FS_Wood`. `SC_FS_Grass` sits in the folder but the footstep notify does not read it.

So copy the closest asset that exists, not the one that happens to share a name with your surface.

---

## Step 1: declare the surface

1. Open **Project Settings**, then **Engine**, then **Physics**.
2. Under **Physical Surface**, add an entry at the end of the list. It takes the next free slot, `SurfaceType7`.
3. Type `Sand` as its name.
4. Save.

Add at the end. Renaming an entry that is already in use changes what every existing `PM_` asset means.

---

## Step 2: create the physical material

1. In `Content/TheLastTemplate/Materials/PhysicsMaterials/`, right click, then **Physics**, then **Physical Material**. Name it `PM_Sand`.
2. Set `Surface Type` to `Sand`.
3. Save.

That one field is the whole job. The friction and density fields on a physical material are physics, and nothing on this page reads them.

---

## Step 3: put it on your mesh

!!! warning "It goes on the mesh, not on the material"
    A physical material assigned to a **Material** asset is ignored by simple traces, and the bullet trace is a simple trace. You get the Default entry instead, with no error and no warning. Put it on the mesh component's collision.

1. Select the mesh component, in the level or inside its Blueprint.
2. In the **Collision** category, set `Phys Material Override` to `PM_Sand`.

The six panels in the firearms area of the demo map are set up exactly this way, one per surface, if you want something to compare against.

---

## Step 4: duplicate an impact effect

1. In `Content/TheLastTemplate/Niagara/Impacts/`, duplicate the closest system. For sand, `NS_Impact_Concrete` is the nearest start. Name it `NS_Impact_Sand`.
2. Open it and change the colours, the counts and the lifetimes to suit.
3. Leave its Effect Type on `NET_Impacts`. That is where the shared budget and scalability settings sit.

Every impact system is built from the same shared emitters: `NE_Debris`, `NE_Dust`, `NE_Flash` and `NE_Spark`, in `Content/TheLastTemplate/Niagara/Shared/Emitters/`. Change values inside your own system. If you open a shared emitter and change it there, every impact in the game changes with it.

---

## Step 5: duplicate a decal instance

1. In `Content/TheLastTemplate/Materials/Effects/Decals/Instances/`, duplicate `MI_Decal_Impact_Concrete` and name it `MI_Decal_Impact_Sand`.
2. Leave its parent on `M_Decal_Impact`.
3. Override the parameters you need.

| Parameter | What it does |
|---|---|
| `Atlas Row` | Picks which row of the impact sheet this surface uses. One row is one look of damage. |
| `Atlas V Scale` | How tall one row is in the sheet. Only change it if you also change `Normal Map`. |
| `Surface Tint` | Colour pushed into the hole. This is the parameter that does most of the work. |
| `Rough Damaged` | Roughness of the damaged area, against the surface around it. |
| `Opacity Mul` | Overall strength of the decal. |
| `Normal Map` | The sheet itself. The default is the shared `T_Decal_Impact_N_4x4`. |
| `Normal Strength` | How deep the hole reads. |

The blood instance is the one to look at if you want your own sheet: it points `Normal Map` at `T_Decal_Blood_N_4x1` instead of the shared one, and it has to change `Atlas V Scale` to match, because that sheet has one row instead of four.

---

## Step 6: build the footstep cue

1. Duplicate a folder under `Content/TheLastTemplate/Audios/Footsteps/`, for example `Wood`, and name the copy `Sand`.
2. Import your takes into it as `WAV_FS_Sand_01` and so on. Several short takes, not one.
3. Duplicate `SC_FS_Wood` into it as `SC_FS_Sand`, open it, and point its **Random** node at your new waves.
4. Leave `ATT_Footstep` as the attenuation and `SC_TLT_Effects` as the sound class, so your surface follows the same falloff and the same mix slider as the others.

---

## Step 7: wire it into the two lookups

Only two assets in the whole project ask what surface was hit.

**`Content/TheLastTemplate/Blueprints/ActorComponents/BP_WeaponManager`**, for anything a bullet does. Three functions, one **Select** node each, with one pin per declared surface:

| Function | What you plug in |
|---|---|
| `Spawn Impact VFX` | `NS_Impact_Sand` |
| `Spawn Decal Impact` | `MI_Decal_Impact_Sand` |
| `Spawn Impact SFX` | the sound for a bullet landing on sand |

Once the surface is declared in step 1, a new pin for it appears on all three Select nodes on its own. Plug your three assets in and compile.

!!! note "The impact sounds are placeholders"
    `Spawn Impact SFX` ships playing the footstep cues at a raised pitch, on purpose, so that every surface makes some sound. When you have real impact audio, plug it in there and drop the `Pitch Multiplier` on the Spawn Sound at Location node back to 1.

**`Content/TheLastTemplate/Animations/AnimNotifies/Character/AN_FootstepSound`**, for footsteps. Same shape: one Select over the surface, feeding the cue that gets played. Plug `SC_FS_Sand` into your pin.

Edit the parent only. `AN_FootstepSound_L` and `AN_FootstepSound_R` are children of it with nothing of their own, and they are what is actually placed on the animations.

That same notify also reports a noise event for the AI to hear, sized by how fast the character is moving through `Walk Noise Range` and `Run Noise Range`. It does not depend on the surface, so a new surface adds nothing there.

## Things that go wrong

- **You get the Default effect everywhere.** The physical material ended up on the Material asset instead of the mesh component. This is the one that costs the most time, because nothing reports it.
- **Effect and sound are right, no decal.** `Spawn Decal Impact` is a separate function from `Spawn Impact VFX`. All three have to be filled.
- **Every impact in the game changed.** You edited a shared emitter in `Niagara/Shared/Emitters/` instead of a value inside your own system.
- **Your decal shows the wrong part of the sheet.** `Atlas Row` and `Atlas V Scale` have to agree with the sheet in `Normal Map`.
- **The footstep is silent on your surface.** Check the parent notify, not `AN_FootstepSound_L` or `AN_FootstepSound_R`.

---

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
