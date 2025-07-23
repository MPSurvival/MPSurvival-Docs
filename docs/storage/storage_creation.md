# 🚀 How to create a storage ?

### 1. Add the storage component

In your Blueprint, add the component `BP_StorageSystem`.

---

### 2. Implement the storage interface

- Add the interface `BPI_StorageManagerInterface` to your Blueprint.
- Implement the interface function `GetStorageComponent`.
- Return your `BP_StorageSystem` component in the return value.

This works like the vital manager interface.

---

### 3. Open the storage UI

To open the storage, call the function `Owning_OpenStorage` from the server side.

![How to open storage](../assets/storage/Storage_01.jpeg)

---

!!! tip  
    Make sure your Blueprint is properly set up on the server to allow multiplayer interaction with storage.

---
