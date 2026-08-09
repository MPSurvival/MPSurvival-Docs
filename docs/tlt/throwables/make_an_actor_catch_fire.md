# Make an actor catch fire

At the end of this page your own prop or character burns when a Molotov pool reaches it: flames on its mesh, damage over time, and it puts itself out on its own. It costs one actor tag, plus one checkbox if the thing burning is a static mesh.

---

## The rule: one tag

`BP_MolotovZone` is the pool of fire a Molotov leaves on the ground. It only touches actors that carry the `CanBeFire` actor tag. When it finds one, it adds a `BP_FireComponent` to that actor, writes the numbers of the burn onto it, and calls `Start Fire`.

That is the whole switch. You do not change the parent class of your actor, and you never place `BP_FireComponent` by hand: it is added at runtime, to whatever was standing in the fire.

- The component: `Content/TheLastTemplate/Blueprints/ActorComponents/BP_FireComponent`
- The pool: `Content/TheLastTemplate/Blueprints/Environments/Zones/BP_MolotovZone`

---

## Which actors already carry the tag

Five actors ship with `CanBeFire` on them.

| Actor | What it is |
|---|---|
| `BP_PlayerCharacter` | The player |
| `BP_NPCCharacter` | The human AI |
| `BP_ZombieCharacter` | The infected AI |
| `BP_Dummy` | The full test character, with vitals and ragdoll |
| `BP_TrainingDummy` | The torso on a stand |

The player is in that list on purpose. Throw a Molotov at your own feet and you burn like anything else.

No environment prop ships with the tag, so a crate, a car or a fuel barrel does nothing when a Molotov lands on it until you add it yourself.

---

## Add the tag to your actor

1. Open the Blueprint of the actor you want to burn, then click `Class Defaults`. For a single actor already placed in a level, select it there instead and skip to step 2.
2. In `Details`, find the `Actor` section and its `Tags` array. Typing `Tags` in the search box is quicker.
3. Add an element and type `CanBeFire`.
4. Compile and save.

The tag is compared exactly, so `canbefire` or `CanBeFired` will never match. Put it on the Blueprint and every copy in every level inherits it. Put it on a placed actor and only that one burns.

---

## Static meshes: tick Allow CPU Access

`Start Fire` spawns one fire per mesh on the actor. A skeletal mesh component gets `NS_Fire_MeshAttach_Skeletal`, anything else gets `NS_Fire_MeshAttach_Static`. Both of those systems place their particles on the surface of the mesh they are attached to, and reading the surface of a static mesh at runtime is off by default in Unreal.

1. Double click the static mesh asset your actor uses.
2. Type `CPU` in the `Details` search box.
3. Tick `Allow CPU Access`.
4. Save.

!!! warning "This one fails without a single message"
    A static mesh with `Allow CPU Access` unticked still receives the fire component and still takes the fire damage. It just never shows a flame, and nothing is logged. If an actor burns invisibly, check this before anything else.

**A prop that burns only in places is almost always this.** An actor built from several static mesh components gets one fire per component. Tick the box on the frame of a car and not on its doors, and the car burns with the doors untouched. Go through every static mesh the actor uses, not only the main one.

A skeletal mesh needs nothing. `NS_Fire_MeshAttach_Skeletal` reads the skinned surface directly, so a character catches fire as soon as it has the tag.

---

## What the fire does once it catches

These are the fields on `BP_FireComponent`, with the values it ships with.

| Field | What it does | Ships at |
|---|---|---|
| `Fire System Skeletal` | Effect spawned on a skeletal mesh component | `NS_Fire_MeshAttach_Skeletal` |
| `Fire System Static` | Effect spawned on every other kind of mesh | `NS_Fire_MeshAttach_Static` |
| `Fire Scale` | Pushed into the effect as its `ScaleMultiplier` | `0.7, 0.7, 0.7` |
| `Attach Socket` | Socket the fire attaches to. Left empty, it sits at the mesh origin | empty |
| `Fire Damage` | Damage applied on each tick of the burn | `5` |
| `Damage Delay Loop` | Seconds between two damage ticks | `0` |
| `Auto Stop Delay` | Seconds before the fire puts itself out | `5` |

Three of them are overwritten when a Molotov pool is what lit the fire: `BP_MolotovZone` writes `Fire Damage`, `Damage Delay Loop` and `Auto Stop Delay` onto the component before starting it. So changing the numbers here does not change what a Molotov does. To change that, change `Fire Damage` and `Fire Duration` on the pool, or the `Amount` and `Duration` of the Molotov's payload row. See [How throwables work](how_throwables_work.md).

The fields above are what a fire uses when you start one yourself, by adding `BP_FireComponent` to an actor in your own Blueprint and calling `Start Fire`. `Start Fire` takes no arguments, it reads the component. If you go that route, give `Damage Delay Loop` a real interval before you call it, because the damage runs on a looping timer.

The damage goes through the normal path, so vitals, hit reactions and death behave as usual. See [How damage works](../damage/how_damage_works.md).

---

## Use your own fire effect

Point `Fire System Static` and `Fire System Skeletal` at your own Niagara systems. Two things are expected of whatever you put there.

- It is spawned attached to the mesh component, snapped to it, at `Attach Socket`. Build it to sit at the origin of what it is attached to, not at a world position.
- It needs a Vector user parameter named exactly `ScaleMultiplier`. `Start Fire` writes `Fire Scale` into that parameter by name. A system without it still plays, `Fire Scale` simply stops doing anything.

If your system does not sample the mesh surface, and emits from a shape instead, the `Allow CPU Access` requirement goes away with it. That is a fair trade for small props: the flames no longer follow the silhouette, but they never fail to appear.


---

[Join the Discord](https://discord.gg/EqHCtq38jy)
