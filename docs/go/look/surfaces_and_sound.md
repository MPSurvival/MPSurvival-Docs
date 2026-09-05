# Surfaces, footsteps and sound

Footsteps are chosen from the surface you are standing on, and the surface is declared on the **material**, never on a component.

---

## The five surfaces

Declared in `Config/DefaultEngine.ini`:

| Slot | Name |
|---|---|
| `SurfaceType1` | Concrete |
| `SurfaceType2` | Ground |
| `SurfaceType3` | Metal |
| `SurfaceType4` | Water |
| `SurfaceType5` | Wood |

Each has a physical material in `Materials/Surfaces/` and a folder of sounds in `Audios/Footsteps/`.

**A material sounds like something because it carries a physical material.** `MI_Surface_Concrete` carries `PM_Concrete`, and anything wearing it sounds like concrete. Do not use `PhysMaterialOverride` on a component: there is none in the project, and an override splits how a surface looks from how it sounds the first time somebody swaps a material.

---

## Adding a surface

1. Add a `SurfaceType` line in `Config/DefaultEngine.ini`.
2. **Restart the editor.** A new entry does not appear in a Blueprint switch until you do, and before that the switch shows 62 raw enum entries.
3. Create a `PM_<Name>` physical material and set its surface type.
4. Assign it to the material instances that should sound like it.
5. Import six variations and build an `SC_Footstep_<Name>` cue.
6. Assign the cue to the new slot in the footstep settings.

Only step 1 touches a config file.

---

## Who plays footsteps

| Who | What plays their steps |
|---|---|
| The player | `BP_ViewMotionComponent`, on distance travelled, so the sound stays locked to the head bob and speeds up with sprint |
| Customers and employees | `BP_FootstepComponent`, on anim notifies, so the sound lands with the foot |

If you put notifies on your own walk animation, note that the `L` and `R` markers on the shipped one are **sync markers**, which line two animations up and emit nothing. Real notifies have to be added on top.

---

## Play sounds attached

Every diegetic sound in the project uses **Spawn Sound Attached**, not Play Sound at Location.

A sound played at a location has no owning actor, so the occlusion trace does not ignore the object making it: the till's sound starts inside the counter, the trace hits the counter, and the cash register sounds like it is in the next room. Attaching the sound makes the engine ignore that actor.

---

## Occlusion

Two attenuation assets carry it: `ATT_Footstep` and `ATT_Default`.

| Setting | Value |
|---|---|
| Enable Occlusion | on |
| Trace Channel | **`Visibility`** |
| Low Pass Filter Frequency | `500` Hz |
| Volume Attenuation | `0.4` |
| Interpolation Time | `0.1` s |

Two things that are easy to get wrong:

**Ticking the box does nothing on its own.** It only starts the trace. The two values that produce the effect default to 20000 Hz, above audible, and 1.0, which is no attenuation at all. Set both.

**The channel must be one that pawns ignore.** The trace runs to the listener, which is the camera inside the player capsule. On `WorldStatic` the capsule blocks it and every sound is occluded all the time, indoors and out.

---

## Sound classes

Six, in `Audios/`: `SCLS_Master` with `Effects`, `UI`, `Music`, `Dialogue` and `Cinematics` under it. Everything routes through one of them, so the mix is set in one place.

---

## The two post process looks

Both live in `Materials/PostProcess/` and both are blendables on the post process volume. Set the weight of the one you want to 1 and the other to 0.

**`M_PostProcess_Outline`** draws an ink outline around marked objects, from a `CustomStencil` mask, so it draws silhouettes only and only on objects that ask for it. Four parameters on `MI_PostProcess_Outline`.

**The painterly filter** is an anisotropic Kuwahara in three passes, running after tone mapping. It never reads the G-buffer, so no material or shading model can break it.

If you write your own materials: **a dithered masked material turns black under the outline**, because a dither pattern is a depth discontinuity at every pixel. Use a translucent material or an opaque tint instead.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
