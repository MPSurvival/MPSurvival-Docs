# 🚀 How to create an interaction with an object ?

### 1. Add the interactable interface

In your Blueprint, add the interface `BP_InteractableInterface`.

---

### 2. Implement the interface functions

Fill in the following interface functions:

| Function Name     | Purpose                                                  |
|-------------------|----------------------------------------------------------|
| `GetHighlightMesh`| Returns the mesh to highlight when the player hovers    |
| `GetObjectName`   | Returns the display name of the object                   |

---

### 3. Use interaction events

You can use these Blueprint events to handle interaction logic:

| Event Name        | Description                                             |
|-------------------|---------------------------------------------------------|
| `OnObjectSpawn`   | Custom spawn call — not called automatically at BeginPlay, call it yourself if needed |
| `OnPlayerStopHover` | Triggered when the player stops hovering over the object |
| `OnPlayerHover`   | Triggered when the player starts hovering over the object |
| `OnPlayerInteract`| Triggered when the player interacts with the object    |

---

!!! tip  
    These allow you to create rich, dynamic interactions adapted to your gameplay.

---
