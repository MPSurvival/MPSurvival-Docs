# Change the sound and the mix

Every sound in the template goes through the same three things: a **sound class** that says what kind of sound it is, an **attenuation** that says how it fades with distance, and a **concurrency** that stops the same sound piling up on itself.

Get those three right and a new sound behaves like the ones that ship, including reacting to the volume sliders in the options menu.

Everything lives in `Content/TheLastTemplate/Audios/`.

---

## The sound classes

Six classes, in one hierarchy.

| Class | What goes in it |
|---|---|
| `SC_TLT_Master` | The parent of all the others. Nothing is assigned to it directly. |
| `SC_TLT_Effects` | Weapons, impacts, footsteps, doors, everything in the world. |
| `SC_TLT_Music` | Music. |
| `SC_TLT_UI` | Menu clicks, notifications, the inventory. |
| `SC_TLT_Dialogue` | Voice. |
| `SC_TLT_Cinematics` | Sound played during a scripted scene. |

The hierarchy is the reason the sliders work. The **Master** slider in the options menu moves `SC_TLT_Master`, and because the other five are its children they all follow. A slider for effects moves `SC_TLT_Effects` alone.

!!! warning
    A sound with no class, or with a class outside this tree, ignores every slider in the options menu. It plays at full volume no matter what the player sets. This is the single most common reason a new sound is too loud.

---

## The attenuations

Seven attenuation assets in `Audios/Attenuation/`, one per family of sound.

| Asset | Used by |
|---|---|
| `ATT_Default` | Anything that has no better fit. |
| `ATT_WeaponFire` | Gunshots. |
| `ATT_Impact` | Bullet impacts and melee hits. |
| `ATT_Explosion` | Explosions. |
| `ATT_Footstep` | Footsteps. |
| `ATT_Foley` | Cloth, gear, small body sounds. |
| `ATT_Voice` | Anything coming out of a character. |

Pick the one that matches what your sound is, rather than making a new one. Seven curves that the player learns to read are worth more than twenty that each behave slightly differently.

They matter for more than volume. A sound the AI can hear is a separate system, described in [Noise, and how to use it](../ai/noise_and_distractions.md). Attenuation is what the **player** hears, noise range is what the **AI** hears, and the two are set in different places on purpose.

---

## Concurrency

`SC_TheLastTemplate` is a sound concurrency asset. It caps how many copies of the same sound can play at once and stops the oldest when the cap is reached.

Without it, a shotgun fired into a group of five enemies plays five impact sounds in the same frame and the mix clips. Put your own repeated sounds on it, especially impacts and footsteps.

---

## Add a sound

Say you want a new door to creak.

1. Import your wave into a folder under `Audios/` that matches what it is.
2. Make a Sound Cue from it if you want several variations picked at random. The footstep cues are built exactly that way, one cue holding six waves.
3. On the cue or the wave, set **Sound Class** to `SC_TLT_Effects`.
4. Set **Attenuation** to `ATT_Default`, or `ATT_Foley` if it is a small close sound.
5. Set **Concurrency** to `SC_TheLastTemplate` if it can play several times at once.
6. Point your actor at it.

Steps 3 to 5 are the ones that get skipped, and skipping them is what makes a sound feel bolted on.

---

## Footsteps

Footsteps are picked by the surface under the foot, not by the level. Six cues ship, in `Audios/Footsteps/`:

`SC_FS_Concrete`, `SC_FS_Grass`, `SC_FS_Ground`, `SC_FS_Metal`, `SC_FS_Water`, `SC_FS_Wood`

The step notify on the animation traces down, reads the physical material it lands on, and plays the matching cue. The player, the human enemies and the infected share the same animations, so a change here changes all of them at once.

Two of those cues are not reachable today. `SC_FS_Grass` exists but the notify does not pick it, and the blood and glass surfaces have no footstep cue at all, so they fall back to the default. If you want a footstep on grass, that is where to look first. How surfaces are declared and matched is covered in [How surfaces work](how_surfaces_work.md).

---

## The menu mix

`SM_TLT_MenuMix` is a sound mix pushed when a menu opens and popped when it closes. It ducks the game so the menu is clearly in front, without muting anything.

To change how much the game ducks, open the mix and change the adjuster on `SC_TLT_Effects`. To duck something else as well, add an adjuster for its class in the same asset. Nothing else has to change: the mix is pushed and popped in one place.

---

## Music

No music ships with the template, and that is deliberate. Music is the most personal choice in a project and a placeholder track is worth nothing to you.

What ships is the hook. `BP_PlayerCharacter` has a `Current Combat Music` variable and a `Play Combat Music` function that starts it when the player enters a fight. Point the variable at your own track, give it `SC_TLT_Music` as its sound class, and combat music works.

Leave the variable empty and nothing plays. Nothing breaks either.

---

## Where the sliders are

The volume sliders live in the audio page of the options menu, which is a Data Asset like every other menu page. Adding a slider for a class you added is a row in that asset, not a widget change. See [Add or change a menu page](..

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
