# Put hit reactions on your own character

The characters that ship lose health when they are hit, get pushed around by the impact, and drop to a ragdoll when they run out. This page moves that whole layer onto a `Character` of your own: first on the skeleton the template uses, then on a skeleton of your own.

There is no manager in the level and no interface to implement. It is two components, two damage events, and, on a new skeleton, a list of bone names.

---

## What you need first

- Your class has to be a `Character`. The reaction system works from the character's `Mesh`.
- Your skeletal mesh needs a physics asset. Without bodies there is nothing to push.
- The `Physics Control` plugin has to be enabled. It already is in the template project. If you migrate the content into a project of your own, turn it on in `Edit`, then `Plugins`, and restart the editor.

---

## The components you add

Both live in `Content/TheLastTemplate/Blueprints/ActorComponents/`.

| Component | What it gives your character |
|---|---|
| `BP_VitalsSystem` | Health, stamina and any vital you added. It is the only thing that removes hit points. |
| `BP_HitReactionManager` | The physical reaction to a hit, and the handover to the ragdoll on death. |

A third one is optional: `BP_FallDamageComponent`, covered in [Fall damage, death and the ragdoll](fall_damage_and_death.md). Add it and it works on its own, with nothing to wire.

Add the two from the `Components` tree with `Add`, then the component name. You do not add a `Physics Control Component` yourself: `BP_HitReactionManager` creates its own when the game starts.

---

## Fill in the Vitals array

`BP_VitalsSystem` ships with `Vitals` empty. Every character fills it in for itself, so a character of yours has no health at all until you do.

1. Select `BP_VitalsSystem` in the `Components` tree.
2. In the `Details` panel, add one entry to `Vitals`.
3. Set `Vital Asset` to `Content/TheLastTemplate/Blueprints/DataAssets/Vitals/Childs/DA_Vital_Health`.
4. Set `Current Amount` to the health the character starts with. Nothing fills it in for you at `Begin Play`.
5. Add a second entry with `DA_Vital_Stamina` if your character sprints.
6. Compile and save.

| Field | What it does |
|---|---|
| `Vital Asset` | The Data Asset that holds the name, the type and the maximum of that vital. |
| `Current Amount` | What the character starts with. Healing is capped at the asset's `Vital Max Amount`. |
| `Is Paused Tick` | Stops that vital's continuous drain or refill for this character. |

`BP_PlayerCharacter` carries health and stamina. `BP_NPCCharacter`, `BP_ZombieCharacter` and `BP_Dummy` carry health only. To change the values, or to add a vital of your own, see [Change health and stamina, or add a new vital](health_stamina_and_new_vitals.md).

---

## Wire the two damage events

Both are events on your character, not on the components.

**`Event Any Damage`** is where health comes off. Drag `BP_VitalsSystem` into the graph and call `Remove vital amount`, with `Type` set to `Health` and `Amount to Remove` fed from the damage that came in.

**`Event Point Damage`** is where the reaction comes from. Drag `BP_HitReactionManager` into the graph and call `Hit reaction`, with:

| Pin on `Hit reaction` | Pin on `Event Point Damage` |
|---|---|
| `Bone Name` | `Bone Name` |
| `Direction` | `Shot from Direction` |
| `Location` | `Hit Location` |

The shipped characters do a little more than that on the same two events: they branch on the damage type so that a shot which arrives as point damage is not paid for twice, and they keep the last bone and direction in their own variables. [How a character takes damage](how_damage_works.md) walks through that split. Copying both event chains out of `BP_NPCCharacter` is the fastest way to get it right.

---

## Give the component your mesh

`BP_HitReactionManager` picks up the skeletal mesh it drives at `Begin Play`, and it only recognises the character classes that ship. A class it does not recognise gets no mesh, so the reaction never runs.

This is the one place where you have to edit a shipped graph. Open `BP_HitReactionManager`, and in `Event Begin Play` handle your own character class the same way the shipped ones are handled, so that its `Mesh` reaches the component. Compile and save.

---

## On the shipped skeleton, that is all

`SK_PlayerCharacter`, `SK_NPC_01`, `SK_NPC_02` and `SK_Zombie_01` share one skeleton, `Content/TheLastTemplate/Meshes/Characters/S_HumanoidCharacter`, and one physics asset, `PA_HumanoidCharacter`, in the same folder.

If your character uses that skeleton, the bone names on `BP_HitReactionManager` already match and the physics asset is already assigned. You are done. Play, shoot it, and the body reacts.

---

## On a skeleton of your own

### Give the mesh a physics asset

Right click your skeletal mesh in the `Content Browser` and create a physics asset from it, then check that the mesh's `Physics Asset` field points at the new one. Every bone the limb setup starts from needs a body on it.

You can also just use `PA_HumanoidCharacter` as a reference for how the shipped bodies are shaped and weighted.

### Point the limb setup at your bone names

Select `BP_HitReactionManager` on your character and open `Limb Setup` in the `Details` panel. It has eight entries. Change the `Start Bone` of each to the matching bone in your skeleton, and leave everything else alone.

| `Limb Name` | Shipped `Start Bone` |
|---|---|
| `Head` | `head` |
| `Neck` | `neck_01` |
| `ArmL` | `clavicle_l` |
| `ArmR` | `clavicle_r` |
| `LegL` | `thigh_l` |
| `LegR` | `thigh_r` |
| `UpperTorso` | `spine_02` |
| `LowerTorso` | `spine_01` |

A limb takes its start bone and everything under it that another limb has not already claimed. `LowerTorso` is the only entry with `Include Parent Bone` checked, and that is what pulls the pelvis into it.

The eight names are the hit regions themselves: a hit reports the region of the bone it landed on, and a hit profile can answer differently per region. Change the bones, keep the names.

`Create World Space Controls`, `Create Parent Space Controls` and `Create Body Modifiers` are checked on all eight. Leave them checked.

### Set the pelvis bone name

`Pelvis Bone Name` on the same component is used when the body is handed over to physics. It ships as `Pelvis`, which is the pelvis of the template's skeleton. Bone names are not case sensitive, so the capital in the shipped value does not matter, but the spelling does. Set it to whatever the pelvis is called in your own skeleton.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
