# Surfaces, footsteps and sound

Footsteps are chosen from the surface you are standing on, and the surface is declared on the **material**, never on a component.

---

## The five surfaces

They are declared in `Config/DefaultEngine.ini`:

| Slot | Name |
|---|---|
| `SurfaceType1` | Concrete |
| `SurfaceType2` | Ground |
| `SurfaceType3` | Metal |
| `SurfaceType4` | Water |
| `SurfaceType5` | Wood |

Each one has a physical material in `Materials/Surfaces/` and a folder of sounds in `Audios/Footsteps/`.

**Adding a surface takes a restart.** A new entry in that ini does not appear in a Blueprint switch until the editor is restarted; before that the switch shows the 62 raw enum entries instead.

---

## Making a material sound like something

Assign the physical material to the **material instance**, not to the mesh component.

There is no `PhysMaterialOverride` anywhere in the project, on purpose. A component override wins over the material and separates how a surface looks from how it sounds, which then goes wrong the first time somebody swaps a material.

So: `MI_Surface_Concrete` carries `PM_Concrete`, and anything wearing it sounds like concrete.

---

## Adding a surface

1. Add a `SurfaceType` line in `Config/DefaultEngine.ini`.
2. Restart the editor.
3. Create a `PM_<Name>` physical material and set its surface type.
4. Assign it to the material instances that should sound like it.
5. Record or import six variations and build an `SC_Footstep_<Name>` cue.
6. Assign the cue to the new slot in the footstep settings.

Steps 4 to 6 are Details panel work. Only step 1 touches a config file.

---

## Two footstep systems, and why

| Who | What plays their steps |
|---|---|
| The player | `BP_ViewMotionComponent.AdvanceFootsteps` |
| Customers and employees | `BP_FootstepComponent`, driven by anim notifies |

The player's steps are **indexed on distance travelled**, not on a timer. A step fires every time the stride phase passes 180 degrees, so the sound is locked to the head bob by construction and speeds up on its own when you sprint. A timer-driven footstep drifts the moment you change pace.

The AI's steps come from notifies on the walk animation, because a character with legs you can see should step when the foot lands. Note that the `L` and `R` markers on the shipped walk are **sync markers**, which line two animations up and emit nothing. Real notifies had to be added on top.

Both paths end in the same place: trace down, read the surface, switch, and play one sound.

---

## The sound is attached, not placed

Every diegetic sound in the project is played with **Spawn Sound Attached**, not Play Sound at Location, and that is a rule rather than a preference.

A sound played at a location has no owning actor, so the occlusion trace does not ignore the object making it. The till plays its sound at the actor origin, which is inside the counter, and the trace hits the counter before it reaches your ears. The result is a cash register that sounds like it is in the next room.

Attaching it makes the component belong to the actor, so the engine ignores that actor in the trace, and the sound comes from the component you point at rather than from the pivot.

Footsteps are attached to the capsule root at zero offset, roughly 88 cm off the floor, rather than at the contact point. Nothing is born inside the floor.

---

## Occlusion

Two attenuation assets carry it: `ATT_Footstep` and `ATT_Default`.

| Setting | Value | Why |
|---|---|---|
| Enable Occlusion | on | |
| Trace Channel | **`Visibility`** | |
| Low Pass Filter Frequency | `500` Hz | |
| Volume Attenuation | `0.4` | |
| Interpolation Time | `0.1` s | |

Two things here are easy to get wrong, and both were:

**Ticking the box does nothing on its own.** It starts the trace. The two values that produce the effect default to 20000 Hz, above audible, and 1.0, which is no attenuation at all. Both have to be set.

**The channel must be one that pawns ignore.** The occlusion trace runs from the source to the listener, and the listener is the camera, which lives inside the player capsule. On `WorldStatic` the capsule blocks the trace, so every sound is occluded all the time, indoors and out. On `Visibility`, pawn capsules and character meshes ignore it by profile and the world blocks it.

---

## Sound classes

Six, in `Audios/`: `SCLS_Master` with `Effects`, `UI`, `Music`, `Dialogue` and `Cinematics` under it.

Everything routes through one of them, so a mix is set in one place.

---

## The look

Two post process materials ship, both in `Materials/PostProcess/`.

**`M_PostProcess_Outline`** draws an ink outline around marked objects. It works on a `CustomStencil` mask rather than on scene depth, so it draws silhouettes only, with no internal edges, and only on the objects that ask for it. It has four parameters on `MI_PostProcess_Outline` and no external dependency.

**A painterly filter** is included as an optional look: an anisotropic Kuwahara filter in three passes, chained through User Scene Textures, running after tone mapping so it is not smeared by temporal anti-aliasing. It never reads the G-buffer, so no material, shading model or blend mode can break it.

Both are blendables in the post process volume. Set the weight of the one you want to 1 and the other to 0.

One thing to know if you write your own: **a dithered masked material turns black under the outline**, because the outline reads a depth discontinuity and a dither pattern is one at every pixel. Use a translucent material or an opaque tint instead.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
