---
order: 500
title: Use New Input System
--- 

This guide walks you through integrating the **Inventory** asset with Unity's **New Input System**.

!!!
This guide uses the `Basic` template as an example. 
!!!


> Prerequisites: 
> - **Input System** package installed.
> - **Input Actions** asset created.


### 1. Define Input Actions

Open the **Input Actions** asset.

Add a new **Action Map** with the following actions:

![](/static/images/tutorials/new-input-asset.jpg){.rounded-lg}

---
### 2. Attach **PlayerInput** component

Select the **Inventory** object in the scene and add a **PlayerInput** component.

!!!
Note the function names in the *Behavior: Send Messages* section. We will use these functions to pass the events down to the **Store**.
!!!

![](/static/images/tutorials/new-input-add-player-input.jpg){.rounded-lg}

---
### 3. Implement a new Input Handling Script

In your `Assets` folder create a new **MonoBehaviour** script and name it `NewInventoryInput`

```cs NewInventoryInput.cs
using UnityEngine;
using GDS.Core.Events;
using GDS.Demos.Basic;
using GDS.Demos.Basic.Events;

public class NewInventoryInput : MonoBehaviour {
    void OnToggleInventory() => Store.Bus.Publish(new ToggleInventory());
    void OnToggleCharacterSheet() => Store.Bus.Publish(new ToggleCharacterSheet());
    void OnCloseInventory() => Store.Bus.Publish(new CloseInventory());
}
```

---
### 4. Replace **InventoryInput** script

Select the **Inventory** object in the scene, remove the old **InventoryInput** script and add **NewInventoryInput**.

![](/static/images/tutorials/new-input-add-script.jpg){.rounded-lg}