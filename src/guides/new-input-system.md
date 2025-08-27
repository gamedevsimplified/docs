---
order: 500
title: New Input System
--- 

The project has been fully migrated to the **New Input System**, it is now a dependency. 

This is a step towards future-proofing the asset and eventual controller support.

### Installation

When installing or updating the **Inventory Framework** asset, you will be prompted to install the **Input System** package.

![](/static/images/tutorials/new-input-step-1.jpg){.rounded-lg}

If you are using Unity 6, chances are the package is already installed and you can skip this step.

If **Input System** is not installed, you can still skip and manually install the latest package version.

![](/static/images/tutorials/new-input-step-1a.jpg){.rounded-lg}

If you choose not to skip, right after it installs the dependency, the program will ask to restart itself to enable the new input backends. 

![](/static/images/tutorials/new-input-step-2.jpg){.rounded-lg}

At this point, the **Inventory Asset** has not been yet imported and by choosing to restart you will have to start over with the installation/update process.

Instead, you should choose not to restart and configure the input handling manually after importing.

Go to **Edit > Project Settings**, then navigate to **Player > Other Settings > Configuration**, and set **Active Input Handling** to *Input System Package* or *Both*.

![](/static/images/tutorials/input-settings.jpg){.rounded-lg}


### Input Actions Asset

!!!
This section uses the `Basic` template as an example. 
!!!

Most demos include an **Input Actions Asset**.

It comes with 2 action maps: Inventory and Player.

![](/static/images/tutorials/input-actions-asset.jpg){.rounded-lg}

As their names suggest, the mappings should be correctly assigned in the **Player Input** script, for *Inventory* and *Player* objects respectively.

The chosen Behavior is **Send Messages** since it's the easiest to setup. Feel free to use a Behavior that meets your needs.

![](/static/images/tutorials/new-input-map-1.jpg){.rounded-lg}
![](/static/images/tutorials/new-input-map-2.jpg){.rounded-lg}



### Inventory Input Script

The **InventoryInput** script defines methods for each action in the mapping.

The methods publish **events** on the **Store Bus**. These **events** will be picked up and further processed by the **Store**.

```cs Demos/Basic/Scripts/InventoryInput.cs
public class InventoryInput : MonoBehaviour {

    Action<CustomEvent> Publish = Store.Bus.Publish;

    public void OnToggleCharacterSheet() => Publish(new ToggleCharacterSheet());
    public void OnToggleInventory() => Publish(new ToggleInventory());
    public void OnCloseInventory() => Publish(new CloseInventory());
    public void OnHotbarUse1() => Publish(new HotbarUse(0));
    public void OnHotbarUse2() => Publish(new HotbarUse(1));
    public void OnHotbarUse3() => Publish(new HotbarUse(2));
    public void OnHotbarUse4() => Publish(new HotbarUse(3));
    public void OnHotbarUse5() => Publish(new HotbarUse(4));

}
```

The **PlayerMovement** script defines a method to store the movement **InputValue** in a local variable.

```cs Common/Scripts/PlayerMovement.cs
public class PlayerMovement : MonoBehaviour {
    // ...
    public void OnMove(InputValue value) => movement = value.Get<Vector2>();
    // ...
}
```