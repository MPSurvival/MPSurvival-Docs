# Place, switch and make lamps

A lamp is an actor with a `BP_LightComponent` on it. The component owns the six settings of the lamp. It keeps them in sync for every player and drives every Light the actor carries. You can place the shipped lamps, switch any of them from gameplay, or build your own.

- The lamps: `Content/MPFriendslop/Blueprints/Environments/Lights/Childs/`
- The base class: `Content/MPFriendslop/Blueprints/Environments/Lights/BP_LightBase`
- The component: `Content/MPFriendslop/Blueprints/ActorComponents/BP_LightComponent`
- The interface: `Content/MPFriendslop/Blueprints/Interfaces/BPI_Light`

---

## The lamps that ship

| Blueprint | Mesh | Light |
|---|---|---|
| `BP_CeilingLight_01` | `SM_CeilingLight_01`, a `60` x `60` cm panel | One Rect Light |
| `BP_CeilingLight_02` | `SM_CeilingLight_02`, a `120` cm bar | One Rect Light |
| `BP_CeilingLight_03` | `SM_CeilingLight_03`, a `100` cm bar | One Point Light |
| `BP_FloorNeon_01` | `SM_FloorNeon_01` | One Point Light |
| `BP_FloorNeon_02` | `SM_FloorNeon_02` | One Rect Light |
| `BP_FloorNeon_03` | `SM_FloorNeon_03` | Two Point Lights, `Light` and `UpperLight` |

The ceiling lamps have their pivot at the fixing point, so they go straight against the ceiling, at Z `300` on the kit. The floor neons have their pivot at the bottom centre and stand on the floor. Every Light is Movable and casts shadows.

---

## Place a lamp

1. Drag one of the six Blueprints into the level.
2. Select it and click `BP_LightComponent` in the Components panel.
3. Set the fields of `Settings|Lighting`.
4. Press Play to judge the result.

| Field | What it does | Default |
|---|---|---|
| `Is Light Active` | Off hides every Light of the lamp | on |
| `Strength` | Brightness. Each Light gets `Strength` x `200` lumens | `5`, so `1000` lumens |
| `Is Blinking` | The lamp flickers on and off while it is active | off |
| `Blinking Interval` | Length of one on or off phase, in seconds | `1` |
| `Blink Randomness` | `0` gives a steady rhythm. `1` gives the most irregular phases, so lamps drift out of step | `0.2` |
| `Light Color` | Colour of every Light of the lamp | white |

A room of flickering lamps looks best with a short interval and `Blink Randomness` at `1`, so the lamps do not flicker in step. The rooms in `Maps/Modules/` use `0.3` seconds.

!!! warning
    The lamp applies its settings when the game starts, not in the editor. The viewport shows the `Intensity` of the Light component itself, so a level can look black in the editor and be lit in game. If you want the editor to match, also set `Intensity` on the Light component of that lamp to `Strength` x `200`.

When the whole level is too dark or too bright, do not touch the lamps one by one. Change the locked exposure first, then the fog, as described in [How levels are built](how_levels_are_built.md).

---

## Switch a lamp from gameplay

Every lamp implements `BPI_Light`. Its six messages match the six fields: `SetLightActive`, `SetStrength`, `SetBlinking`, `SetBlinkingInterval`, `SetBlinkRandomness` and `SetLightColor`. Each takes one input, `Value`. A power cut, an alarm or a quest step can drive any lamp through the interface without knowing its class.

The same six functions exist on `BP_LightComponent` itself, if you already hold a reference to the component.

| Dispatcher | Fires on | When |
|---|---|---|
| `OnLightStateChanged` | every player | A setting changes, or the lamp flickers on or off |

Bind `OnLightStateChanged` for a buzz sound, a spark effect or a HUD icon. It has no parameters: read the fields of the component to know the new state.

The lamps do not implement `BPI_Activatable` when they ship, so a button or a lever cannot switch them as they are. The recipe to add it is in [Make your own door, switch or lever](../interaction/make_your_own_door_or_lever.md).

---

## Make your own lamp

1. Right click `BP_LightBase`, then **Create Child Blueprint Class**.
2. Add a Static Mesh component with your fixture.
3. Add one or more Light components: Point, Rect or Spot.
4. On each Light, set `Intensity Units` to `Lumens` and `Mobility` to `Movable`.

You do not wire anything. `BP_LightComponent` finds every Light of its actor on its own, so a lamp with three Lights switches all three. The `Lumens` unit matters: it is what makes `Strength` x `200` mean the same brightness on your lamp as on the shipped ones.

For a lens that glows with the lamp, put `MI_LightDiffuser` on the lens slot of your mesh. The component writes `Strength` into the `LightStrength` parameter and `Light Color` into the `EmissiveColor` parameter of every material of the actor, so the lens dims, changes colour and goes dark with the Lights. A material of your own glows the same way if it has those two parameters. The scalar name is `Scalar Parameter Name` on `BP_LightComponent`, in `Settings|Materials`.

The component works on any actor, not only on children of `BP_LightBase`. Add a `BP_LightComponent` to a vending machine or a computer and it drives the lights of that actor. Only children of `BP_LightBase` answer `BPI_Light`, though: on another actor, call the functions of the component, or add the interface yourself.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
