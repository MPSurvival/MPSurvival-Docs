# Add an emote or a face

An emote is one Data Asset. It shows a face on the robot's screen, plays a gesture on the upper body, or both. Players hold `T`, aim at a sector of the wheel and release. By the end of this page you have a new entry in the wheel, and if you want one, a new face.

- The emote class: `Content/MPFriendslop/Blueprints/DataAssets/Emote/BP_EmoteDataAsset`, instances in `Emote/Childs/`
- The face class: `Blueprints/DataAssets/Face/BP_FaceDataAsset`, instances in `Face/Childs/`
- The face states: `Blueprints/Enumerations/E_FaceState`
- The two components, both on `BP_FriendslopCharacter`: `BP_EmoteComponent` and `Face` (a `BP_FaceComponent`)

---

## The emotes that ship

| Asset | Emote Name | Emote Montage | Face State | Changes Face |
|---|---|---|---|---|
| `DA_Emote_Neutral` | NEUTRAL | none | `Neutral` | true |
| `DA_Emote_Happy` | HAPPY | none | `Happy` | true |
| `DA_Emote_Sad` | SAD | none | `Sad` | true |
| `DA_Emote_Angry` | ANGRY | none | `Angry` | true |
| `DA_Emote_Surprised` | SURPRISED | none | `Surprised` | true |
| `DA_Emote_No` | NO | `AM_Emote_No` | `Neutral` | false |
| `DA_Emote_Yes` | YES | `AM_Emote_Yes` | `Neutral` | false |
| `DA_Emote_Wave` | WAVE | `AM_Emote_Wave` | `Neutral` | false |

The first five are faces only: a face stays on until the player picks another one. The last three are gestures that leave the face alone and end with their montage. None of them loops.

---

## Add an emote

1. Duplicate `DA_Emote_Happy` for a face emote, or `DA_Emote_Wave` for a gesture. Keep it in `Emote/Childs/`.
2. Set `Emote Name` and `Emote Icon`.
3. For a gesture, set `Emote Montage`. For a face, set `Face State` and tick `Changes Face`.
4. Open `BP_FriendslopCharacter`, select `BP_EmoteComponent` in the Components panel, and add your asset to `Emotes`.
5. Save, play, hold `T`.

The order of `Emotes` is the order of the sectors. The wheel has 8 sectors and the shipped list fills them. A ninth entry opens a second page. While the wheel is open, the mouse wheel turns the pages. The page counter appears on its own.

| Field | What it does |
|---|---|
| `Emote Name` | The text in the centre of the wheel when the sector is aimed at |
| `Emote Icon` | The sector icon. A white shape on transparent, tinted by the wheel. Copy the settings of `T_Icon_Emote_Happy` |
| `Emote Montage` | Played on every player's machine. Empty means a face only emote |
| `Face State` | The face shown while the emote runs |
| `Changes Face` | True: the emote is a face. It shows `Face State` and stays until another pick. Moving or taking damage does not cancel it. False: the emote is a gesture. It never touches the face, and moving or taking damage cancels it |
| `Loops Montage` | False: the emote ends when the montage ends. True: the emote does not end with the montage and runs until it is cancelled. Use it for a dance, with the montage section looping on itself |

---

## The gesture montage

1. Make an `AM_` from your animation on the mannequin skeleton. The shipped ones are in `Animations/Montages/`.
2. Put it on the `UpperBody` slot. Only the spine and above play the emote, so the legs keep their locomotion pose.
3. If the gesture uses the hands, add `BP_HandIKSuspendNotifyState` over the part where they move. Without it, a player who holds an item keeps their left hand glued to the grip while the arm plays the gesture. The three `AM_Emote_*` montages have it.

---

## Add a face

A face is a state in `E_FaceState` plus one Data Asset that draws it. The six states that ship are `Neutral`, `Happy`, `Sad`, `Angry`, `Surprised` and `Dead`.

1. Open `E_FaceState` and add an entry, for example `Scared`.
2. Import your textures: one eyes image and the mouth frames. The shipped ones are in `Textures/Characters/Faces/`. They are pixel art on a 66 x 50 grid, with `Filter` on `Nearest`, no mipmaps and the `UI` texture group. Copy the settings of `T_Face_Eyes_Happy`.
3. Duplicate `DA_Face_Happy` in `Face/Childs/` and fill it in.
4. Open `BP_FriendslopCharacter`, select `Face`, and add your asset to `Faces`.
5. Make an emote that uses it: `Face State` on your new entry, `Changes Face` ticked.

| Field | Where | What it does |
|---|---|---|
| `State` | Face Data Asset | Which `E_FaceState` entry this asset draws |
| `Eyes` | Face Data Asset | The eyes image, white on transparent. It is tinted |
| `Mouth Frames` | Face Data Asset | Mouth images from closed (first) to wide open (last) |
| `Faces` | `Face` component | The faces this screen can draw. It draws the one whose `State` matches |
| `Face Color` | `Face` component | The tint of the eyes and the mouth |

The open mouth frames are for talking. `BP_FaceComponent` has `Set Mouth Openness` (0 to 1) for a voice chat plugin to call on each machine. Nothing in the template calls it.

A face does not have to come from an emote. Call `Server_SetFace` on the `Face` component from your own server-side code, for example a scared face when an enemy sees the player.

---

## The fields on the components

| Field | Component | What it does | Default |
|---|---|---|---|
| `Emotes` | `BP_EmoteComponent` | The wheel content, in sector order. It is also the list of emotes the server accepts | the 8 `DA_Emote_*` |
| `Emote Wheel Class` | `BP_EmoteComponent` | The wheel widget | `WBP_EmoteWheel` |
| `Move Cancel Speed` | `BP_EmoteComponent` | Moving faster than this, in cm/s, cancels a gesture. Faces are not cancelled | `10` |
| `Resting Face` | `BP_EmoteComponent` | The face set when the player releases on the centre of the wheel to stop a face emote | `Neutral` |
| `Wheel Context` | `BP_EmoteComponent` | The mapping context added while the wheel is open | `IMC_EmoteWheel` |
| `Wheel Context Priority` | `BP_EmoteComponent` | Above `IMC_Default`, so the mouse wheel turns pages instead of inventory slots | `1` |

A gesture also stops when the player's health drops below what it was when the gesture started. Health is read through `BPI_VitalManagerInterface`. With your own health system, call `Server_StopEmote` from your damage code.

---

## The dispatchers

| Dispatcher | Component | When it fires | Fires on |
|---|---|---|---|
| `OnEmoteChanged` | `BP_EmoteComponent` | An emote started or stopped. Gives the `Emote` | every player |
| `OnWheelToggled` | `BP_EmoteComponent` | The wheel opened or closed. Gives `Open` | owning player |
| `OnFaceChanged` | `Face` | The face changed. Gives `New Face` | every player |

---

Next: [Player colours, nameplates and pings](colours_nameplates_and_pings.md).

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
