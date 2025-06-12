---
order: 800
title: Add new items 🚧
--- 

{{include "snippets/wip"}}

## Add new item base

> manual process

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

## Add new initial item

> manual process

```cs #3 InitialState.cs
public static BasicItem[] Hotbar = new[] {
    Bases.HealthPotion.Create(20),
    Bases.ManaPotion.Create(20), /*✚*/
};
```
![The Mana Potion should appear in the Hotbar](/static/images/tutorials/add-item-hotbar.jpg){.rounded-lg}

## Add a store component to a scene object

> works only with `Basic`

> when working on original `Basic` template
 
1. Remove `StoreBag`
2. Add `WorldBag`
3. Specify an Id
4. Set size and add items

> when working with a cloned `Basic` template, add `ItemDrawer` editor script for better experience

5. Copy `ItemDrawer.cs` from `Basic/Editor` to `Game/Editor`
6. Rename the namespace from `Basic` to **your chosen namespace**