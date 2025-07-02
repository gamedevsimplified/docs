---
order: 800
title: Add new items
--- 

## Add new item bases

New item types can be added by defining additional fields in the `Bases.cs` file. 

Each field represents a new item base that becomes part of the system.

For now, this is a manual process, but in the future it may be driven by a **ScriptableObject** or a server request.

```cs # Bases.cs
public static class Bases {

    /*...*/

    public static readonly ItemBase ManaPotion = new() {
        Id = "ManaPotion",
        Name = "Mana Potion",
        Icon = "Shared/Images/items/mana",
        Stack = new(10),
        Class = Class.Consumable
    };
}
```

## Add new initial items

To add a new initial item to an inventory, update the appropriate variable in `Store/InitialState.cs`.

For example, the following adds a stack of 20 **Mana Potions** to the **Hotbar**.

The initial state is applied when the inventory is first initialized or reset.

```cs #3 InitialState.cs
public static BasicItem[] Hotbar = new[] {
    Bases.HealthPotion.Create(20),
    Bases.ManaPotion.Create(20), /*✚*/
};
```
![The Mana Potion should appear in the Hotbar](/static/images/tutorials/add-item-hotbar.jpg){.rounded-lg}

## Add a Store Component to a GameObject

The Basic template allows you to attach an inventory to a **GameObject** and add items directly in the editor.

In this example, we define a **Chest** containing 3 items:
 
1. Select an "interactive object" in the scene.
2. Remove the `StoreBag` script.
3. Add the `WorldBag` script (from `Basic/Scripts`).
4. Specify a *Bag ID* and *Type*.
5. Set *size* and add *items*.

![Add items to a Chest](/static/images/tutorials/world-bag.jpg){.rounded-lg}

<!-- If you are working with a cloned `Basic` template, add `ItemDrawer` editor script for better experience

1. Copy `ItemDrawer.cs` from `Basic/Editor` to `[YourGame]/Editor`
2. Rename the namespace from `Basic` to **your chosen namespace** -->