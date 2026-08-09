# Change the visual effects

Fifteen Niagara systems ship with the template, and almost none of them is built from scratch. They share seven emitters, eight master materials and one library of material instances. Changing how a bullet hit, a muzzle flash or a fire looks is mostly a matter of picking the right layer to edit, because each layer has a different blast radius.

Everything lives in three folders:

- The systems and the emitters: `Content/TheLastTemplate/Niagara/`
- Their materials: `Content/TheLastTemplate/Materials/Effects/Niagara/`
- Their textures: `Content/TheLastTemplate/Textures/Effects/Niagara/`

---

## What ships, and what plays it

| System | Folder | Played by |
|---|---|---|
| `NS_Impact_Blood` | `Niagara/Impacts/` | `BP_WeaponManager` when a bullet lands, chosen by surface |
| `NS_Impact_Concrete` | `Niagara/Impacts/` | same |
| `NS_Impact_Grass` | `Niagara/Impacts/` | same |
| `NS_Impact_Metal` | `Niagara/Impacts/` | same |
| `NS_Impact_Water` | `Niagara/Impacts/` | same |
| `NS_Impact_Wood` | `Niagara/Impacts/` | same |
| `NS_Muzzle_Pistol` | `Niagara/Muzzle/` | the gun's Data Asset, through `Muzzle Flash Effect` |
| `NS_Muzzle_Shotgun` | `Niagara/Muzzle/` | same, on both shotgun Data Assets |
| `NS_Shotgun_Rack` | `Niagara/Muzzle/` | `AN_Shotgun_Pump`, an animation notify |
| `NS_Fire_MeshAttach_Static` | `Niagara/Fire/` | `BP_FireComponent`, on a static mesh component |
| `NS_Fire_MeshAttach_Skeletal` | `Niagara/Fire/` | `BP_FireComponent`, on a skeletal mesh component |
| `NS_Fire_MolotovBurst` | `Niagara/Fire/` | `BP_MolotovZone`, the pool of fire on the ground |
| `NS_Fire_Simple` | `Niagara/Fire/` | `BP_Cosmetic_Molotov`, the lit bottle in the hand |
| `NS_Explosion_Grenade` | `Niagara/Explosion/` | `BP_ExplosionZone` |
| `NS_NailBomb_Shrapnel` | `Niagara/NailBomb/` | `BP_NailsZone` |

Which surface picks which impact system, and where that choice is made, is [How surfaces work](how_surfaces_work.md). The three zone actors are covered in [Place hazards and traps](../throwables/place_hazards_and_traps.md).

---

## The four layers

An effect you see on screen is four assets deep. Edit the shallowest layer that gets you what you want.

| Layer | Where | What changes if you edit it |
|---|---|---|
| System `NS_*` | `Niagara/` | That effect only. This is the safe layer. |
| Emitter `NE_*` | `Niagara/Shared/Emitters/`, `Niagara/Fire/Emitters/` | Every system built on that emitter. |
| Material instance `MI_*` | `Materials/Effects/Niagara/` | Every emitter that renders with it. |
| Master material `M_*_Master` | same folders | Every instance under it. |

Four emitters are shared across impacts and muzzles: `NE_Debris`, `NE_Dust`, `NE_Flash` and `NE_Spark`. Three more sit behind the fires: `NE_Fire_Body`, `NE_Fire_Licks` and `NE_Smoke_Column`. Open one of those and change a value there, and every effect that uses it moves with you. Sometimes that is exactly what you want. Usually it is not, and the value you wanted is already overridden inside the system.

`NE_Impact_Debris` sits next to the fire emitters and nothing uses it. It is a spare.

Three master materials carry the shared look: `M_Dust_Master` for anything soft and lit, `M_Additive_Master` for anything that glows, `M_Debris_Master` for lit chunks with a cutout. Five more are specific: `M_Fire_Master`, `M_Fire_Smoke_Master`, `M_Fire_Embers_Master`, `M_Fire_HeatHaze_Master`, `M_Explosion_Master`.

The surface variants under `Materials/Effects/Niagara/Shared/Instances/` are thinner than they look. `MI_Dust_Wood`, `MI_Dust_Concrete`, `MI_Debris_Metal` and their siblings mostly override one colour, `Dust Tint` or `Base Tint`, and nothing else. So making wood debris redder is one field on one instance.

---

## Retune a system without opening its graph

Most of the systems expose their useful values in the `User Parameters` panel of the Niagara editor. Those values are defaults: unless something writes over them when the system is spawned, the default is what plays. Changing one there and saving is the whole edit.

The same parameters appear under `Override Parameters` on any Niagara component you place in a level, so you can give one placed fire a different size without touching the asset.

**Impacts.** All six expose `ImpactStrength` and `DustTint`. On top of that:

| System | Also exposes |
|---|---|
| `NS_Impact_Blood` | `BloodTint`, `DebrisTint` |
| `NS_Impact_Concrete` | `DebrisTint` |
| `NS_Impact_Grass` | `DebrisTint` |
| `NS_Impact_Metal` | `SparkTint`, `VapourTint`, `FlashIntensity` |
| `NS_Impact_Water` | `WaterTint`, `JetDelay`, `BackVector` |
| `NS_Impact_Wood` | `DebrisTint`, `FreshBreak` |

**Fire.** The four fire systems share one set of parameters, so retuning any of them is the same handful of fields.

| Parameter | What it does |
|---|---|
| `FlameAmount` | How much flame |
| `SmokeAmount` | How much smoke |
| `EmberAmount` | How many rising embers |
| `HeatHazeAmount` | Strength of the refraction over the flame |
| `LightIntensity` | Brightness of the light the fire casts |
| `ScaleMultiplier` | Overall size of the fire |
| `SpawnRateScale` | Density, without changing the size |
| `Irregularity` | How much the flame wanders |
| `Wind` | Direction and strength the flame leans in |

`NS_Fire_MeshAttach_Static` and `NS_Fire_MeshAttach_Skeletal` add `SurfaceOffset`, how far off the skin the flame sits. `NS_Fire_MolotovBurst` adds `BurstSpread`, `BurstStrength` and `PoolGrowTime`, which are the shape of the first second after the bottle breaks.

**Muzzle.** `NS_Muzzle_Pistol` exposes `FlashScale` and `LightScale`. `NS_Muzzle_Shotgun` adds `DebrisTint`. `NS_Shotgun_Rack`, the puff of smoke when the pump is worked, exposes `SmokeScale` and `DustTint`.

`FlashScale` is not the only thing sizing a muzzle flash. `Muzzle Flash Scale` on the gun's Data Asset scales the spawned system on top of it, and that is the one set per gun. Change it there before you touch the system. See [Change how a gun feels to shoot](../weapons/change_how_a_gun_feels.md).

**Nail bomb.** `NS_NailBomb_Shrapnel` is the one system whose particles are a real mesh: it renders `SM_Nail` with `MI_Prop_Nail`. It exposes `NailCount`, `NailScale`, `NailLifetime`, `LaunchSpeed` and `SpreadRadius`. Raising `NailCount` costs more than raising a sprite count, because each nail is drawn geometry.

`NS_Explosion_Grenade` exposes nothing. It is tuned inside the system.

---

## Impacts, and the emitters behind them

The six impact systems are not all built the same way, so pick the right one to copy.

| System | Emitters it uses |
|---|---|
| `NS_Impact_Concrete` | `NE_Debris`, `NE_Dust`, `NE_Flash`, `NE_Spark` |
| `NS_Impact_Metal` | `NE_Dust`, `NE_Flash`, `NE_Spark` |
| `NS_Impact_Water` | `NE_Dust`, `NE_Spark` |
| `NS_Impact_Wood` | `NE_Debris`, `NE_Dust` |
| `NS_Impact_Grass` | `NE_Debris`, `NE_Dust` |
| `NS_Impact_Blood` | `NE_Debris`, `NE_Dust` |

Each of them then overrides the material on those emitters with the variant for its surface. Wood uses `MI_Debris_Wood` and `MI_Dust_Wood`, grass uses `MI_Debris_Grass`, `MI_Debris_Soil`, `MI_Dust_Grass`, water uses `MI_Splash`, `MI_Foam`, `MI_RippleRing` and `MI_Droplet_Water`, blood uses `MI_Blood_Sheet` and `MI_Droplet_Blood`. That override is per emitter inside the system, so it never leaks to another surface.

The effects are deliberately thin. The counts and the opacities are set low, well under what a Niagara tutorial would tell you, so that a firefight does not turn the screen into soup. If you want more, raise the counts and the alpha before you raise the sizes: bigger particles read as fog much faster than more of them.

---

## Fire

Four systems, one look. `NS_Fire_MolotovBurst` is the ground pool, `NS_Fire_Simple` is the bottle in the hand, and the two `MeshAttach` systems paint fire onto whatever mesh they are attached to. They are not assembled the same way: the Molotov burst uses `NE_Fire_Body`, `NE_Fire_Licks` and `NE_Smoke_Column`, the two mesh systems use `NE_Fire_Body` and `NE_Smoke_Column`, and `NS_Fire_Simple` carries its own emitters. What makes them look like one family is the materials, which all four share.

The look comes from `M_Fire_Master` and its instances in `Materials/Effects/Niagara/Fire/`. It is not a flipbook. The flame is scrolling noise carved by a mask, read from `T_Fire_NoisePack_01`, `T_Fire_MaskPack_01` and `T_Fire_Curl_01`. The parameters worth touching are `NoiseTilingA`, `NoiseTilingB`, `NoiseTilingC`, `HeightErosion`, `TemperatureFloor` and `TemperatureOffset`, all six of them already overridden on `MI_Fire_Body` and `MI_Fire_Licks`. There is no colour picker. The material turns a temperature into a colour through `T_Fire_LUT_Blackbody_01`, so a colder or a hotter flame is `TemperatureFloor` and `TemperatureOffset`.

Four material functions in `Materials/Functions/Niagara/Fire/` do the shared work: `MF_Fire_ScrollingUV`, `MF_Fire_Erosion`, `MF_Fire_TemperatureLUT` and `MF_Fire_SoftDepthFade`, plus `MF_Fire_CameraDistanceFade` used by the heat haze. The smoke has its own master, `M_Fire_Smoke_Master`, with `SmokeColor`, `SootAmount`, `Density`, `SelfGlowStrength` and `SelfGlowFalloff` overridden on `MI_Fire_Smoke_Column`. The shimmer is `M_Fire_HeatHaze_Master`, with `RefractionStrength`, `HazeSpeed` and `HazeTiling`. The rising sparks are `M_Fire_Embers_Master`.

To set your own props on fire, and for the one checkbox a static mesh needs before a flame will appear on it, see [Make an actor catch fire](../throwables/make_an_actor_catch_fire.md).

---

## Explosion

`NS_Explosion_Grenade` is self contained: it borrows no emitter from anything else. Four material instances make it, and they do not all sit on the same master.

| Part | Material | Sheet |
|---|---|---|
| Fireball | `MI_Explosion_Fireball` on `M_Explosion_Master` | `T_Fireball_8x8` |
| Smoke | `MI_Explosion_Smoke` on `M_Dust_Master` | `T_SmokeBillow_8x8` |
| Dust wave | `MI_Explosion_DustWave` on `M_Dust_Master` | `T_SmokeBillow_8x8` |
| Shockwave | `MI_Explosion_Shockwave` on `M_Fire_HeatHaze_Master` | `T_Shockwave_Ring` |

`M_Explosion_Master` has no tint parameter, on purpose. The fireball's sheet carries a heat value, which is read through `Temperature LUT`, the same `T_Fire_LUT_Blackbody_01` the fire uses. `Heat Scale` and `Heat Contrast` decide where on that ramp the flame lands, and `Soot Color` is the only flat colour in it. So a redder explosion is `Heat Scale`, and a dirtier one is `Soot Color`.

The smoke sharing `M_Dust_Master` with the impacts is worth knowing before you edit that master to fix an explosion. It would take every dust puff in the game with it.

---

## Decals

One master, `Materials/Effects/Decals/M_Decal_Impact`, and six instances under `Instances/`. They are the holes bullets leave.

The sheet is an atlas. Four of the six instances read the shared pair `T_Decal_Impact_MRA_4x4` and `T_Decal_Impact_N_4x4`, and separate themselves with `Atlas Row`, `Surface Tint`, `Opacity Mul`, `Rough Damaged` and `Specular Mul`. Blood and glass bring their own pair, `T_Decal_Blood_*_4x1` and `T_Decal_Glass_*_4x1`, which have one row instead of four. The full set of parameters on the master is `Atlas Row`, `Atlas V Scale`, `Atlas Rand`, `Surface Tint`, `Opacity Mul`, `Rough Damaged`, `Specular Mul`, `Normal Map`, `MRA` and `Normal Strength`. What each one does, and how to duplicate an instance for a new surface, is in [Add a new surface](add_a_new_surface.md).

How big a hole is, how long it stays and when it fades out are not on the material. They are set when the decal is spawned, in the `Spawn Decal Impact` function on `BP_WeaponManager`, through `Decal Size`, `Life Span` and `Fade Screen Size` on the spawned decal component. That is where you go to make bullet holes persist longer, or stop them piling up.

---

## Blood on characters

The blood you see on a body is not a particle effect and not a decal. It is written into the character material, so it survives animation and follows the skin.

- The material: `Materials/Characters/Colors/M_CharacterColor`
- The functions it calls: `Materials/Functions/MF_BloodImpacts`, `MF_BloodField`, `MF_ZombieDecay`

Two static switches turn the big blocks on and off: `Use Blood Impacts` and `Use Zombie Decay`. Below them the parameters are grouped so you can find them: `1 - Blood Pattern`, `2 - Blood Bands`, `3 - Grime`, `4 - Blood Look`, `5 - Blood Surface`, `6 - Blood Normal` and `Blood Impacts`. The ones that do the most work are `Blood Amount`, `Blood Opacity`, `Blood Wet Color`, `Blood Dried Color`, `Blood Wet Start`, `Blood Dried Start` and `Blood Relief`, plus the `Wound *` set for the deeper marks.

Every character material instance in the project is a child of `M_CharacterColor`: the player, both NPC sets, the companion and the infected. Edit the master and all of them change together. Edit a single `MI_*` under `Materials/Characters/Colors/` if you want one character to look different.

---

## Effect density and distance

Two Niagara Effect Types ship, in `Niagara/Shared/`.

| Asset | Assigned to |
|---|---|
| `NET_Impacts` | the six `NS_Impact_*` systems |
| `NET_Muzzles` | `NS_Muzzle_Pistol`, `NS_Muzzle_Shotgun`, `NS_Shotgun_Rack` |

An Effect Type is where the budget of a family of effects is decided in one place instead of system by system. The fields to look at are `Cull By Distance` with `Max Distance`, `Cull Max Instance Count` with `Max Instances`, `Cull By View Frustum`, `Cull When Not Rendered`, and `Spawn Count Scale` in the scalability settings, which thins a system out on lower quality levels without you making a second version of it.

Point your own system's `Effect Type` at `NET_Impacts` and it inherits all of it. Change a number on `NET_Impacts` and every impact in the game follows.

!!! note "Fire, explosion and the nail bomb have no Effect Type"
    `NS_Fire_*`, `NS_Explosion_Grenade` and `NS_NailBomb_Shrapnel` ship with `Effect Type` empty, so nothing bounds them. They are one-off effects and that is survivable, but if you build a level where twenty props burn at once, making a third Effect Type and assigning it to the fire systems is the cheapest fix there is.

---

## Use your own textures

The rule is the same as everywhere else on this page: duplicate the material instance, do not repoint the master.

| Master | Texture parameter | What ships in it |
|---|---|---|
| `M_Dust_Master` | `Flipbook` | `T_Smoke_8x8`, and `T_DustSheet_4x4` on some instances |
| `M_Additive_Master` | `Sprite` | `T_Spark_Streak`, `T_Ember`, `T_MuzzleCone`, `T_MuzzleFlash_Pistol_4x4` |
| `M_Explosion_Master` | `Flipbook` | `T_Fireball_8x8` |
| `M_Debris_Master` | on the master | `T_Debris_4x4`, shared by every debris variant |

1. Import your sheet next to the one you are replacing, in `Textures/Effects/Niagara/`.
2. Duplicate the material instance that uses the old one, for example `MI_Dust_Wood`.
3. Set its texture parameter to your sheet.
4. Open the system that should use it, select the emitter, and set its renderer material to your instance.

The number in a texture name is its grid. `_4x4` is sixteen frames, `_8x8` is sixty four, `_4x1` is four frames in a single row.

!!! warning "A sheet with a different grid plays garbage, silently"
    The frame count is not read from the texture. It is `Sub Image Size` on the emitter's Sprite Renderer, and for a decal it is `Atlas V Scale`. Drop an 8 by 8 sheet into a slot that was 4 by 4 and everything still compiles, still plays, and shows a quarter of each frame sliding across itself. If a swapped texture looks scrambled, this is it.

`M_Debris_Master` is the exception worth remembering. Its sheet is not a parameter, so `T_Debris_4x4` is the shape of every chunk in the game and the five debris instances only re-tint it through `Base Tint`. If you want wood to break into its own shapes, turn that texture sample into a parameter on the master first. Everything under it keeps the old sheet as its default, and you override it on the one instance you care about.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
