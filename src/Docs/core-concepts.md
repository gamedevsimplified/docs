---
order: "900"
title: Core Concepts
---

The framework is built around several key concepts that define its structure and behavior. Understanding these will help you make the most of its features.

### Item

An **Item** is a data type that represents an individual object in an inventory.

```cs Item.cs
public record Item {
	public IItemBase Base;
	public int Id;
	public int Quant;
}
```

### ItemBase

**ItemBase** contains fixed properties like name, icon path and stacking info.

```cs ItemBase.cs
public class ItemBase : IItemBase {	
	public string Id;
	public string Name;
	public string Icon;
	public Stack Stack;
}
```

This architecture is heavily inspired from RPGs like [Diablo](https://diablo2.io/base/) and [Path of Exile](https://www.poewiki.net/wiki/Body_armour) but the separation ensures adaptability to fit various gameplay scenarios. 


### Slot

A **Slot** is a container that can either hold an **Item** or remain empty. Slots can enforce **restrictions**, such as allowing only specific types of items, by providing a function (`Accepts`) that operates on `Item` type. 

```cs Slot.cs
public record Slot(Item Item) { 
	public Predicate<Item> Accepts = _ => true; 
};
```

The base `Slot` type is extended into specialized slot types: 
- `ListSlot`: Index-based storage.
- `SetSlot`: Named slots for equipment/wearable items.
- `GridSlot` (PRO): Grid-based position storage.

```cs Slot.cs
public record ListSlot(int Index, Item Item) : Slot(Item);
public record SetSlot(string Key, Item Item) : Slot(Item);
public record GridSlot(Pos Pos, Item Item, bool Enabled = true) : Slot(Item);
```


### Bag

The **Bag** represents an inventory. Each inventory type comes with its own behavior and can enforce restrictions on accepted items. 

```cs Bag.cs
public abstract class Bag {
	public Predicate<Item> Accepts = (_) => true;        
};
```

The base `Bag` type is extended into specialized types:
- `ListBag`: Contains a list of `ListSlots`
- `SetBag`: Contains a list of `SetSlots`
- `GridBag` (PRO): Contains a 2D array of `GridSlots` as well as a list of `GridItems`


### Event Bus and Events

The framework uses an [**Event Bus**](https://dzone.com/articles/design-patterns-event-bus) to decouple its components. **Events** are data containers describing actions, such as picking up an item. For example, a `PickItem` event includes the item itself, the source inventory and slot.

```cs Events.cs
public record PickItem(Bag Bag, Slot Slot, EventModifiers Mods);
public record PlaceItem(Bag Bag, Slot Slot, Item Item, EventModifiers Mods);
public record AddItem(Bag Bag, Item Item);
```

An event does not perform any actions itself, that falls under the responsibility of the **Store**.

### Store

Inspired by web technologies like [**Redux**](https://redux.js.org/faq/general#when-should-i-use-redux), the **Store** serves as the central hub for managing system state. It subscribes to events on event channels and performs necessary state updates. All changes to the system state flow through the **Store**, ensuring consistency and predictability.

```cs Store.cs
public Store() {
	Bus.Subscribe<PickItem>(e => OnPickItem(e as PickItem));
	// ...
}

void OnPickItem(PickItem e) {
	LogEvent(e);
	// ...
}
```


### Item Bases

This is your repository of item bases and all the types that fully describe and categorize them.
For example an *Apple* is defined as a **stackable**, **consumable** **basic item**, while a *Dagger* is a **1-handed** **weapon**, with *Attack Damage* and *Attack Speed*

```cs Bases.cs
public static readonly ItemBase Apple = new() { Id = "Apple", Name = "Apple", Icon = "Shared/Images/items/apple", Stack = Stack.Infinite, Class = Class.Consumable };
public static readonly WeaponBase ShortSword = new() { Id = "ShortSword", Name = "Short Sword", Icon = "Shared/Images/items/sword-blue", Class = Class.Sword1h, Attack = new(80, 100), AttackSpeed = 1.5f };
```

### Behaviors

The UI is decoupled from any interactivity logic. All logic is abstracted away in behaviors, like showing a tooltip on hovering an item, dragging an item to pick it up or showing a "ghost" version of the dragged item.
```cs
// Behaviors.cs
public static VisualElement WithDragToPickBehavior(this VisualElement element, Observable<Item> draggedItem, EventBus bus) {/*...*/}
public static VisualElement WithItemTooltipBehavior<T>(this VisualElement root) where T : Component<Item> {/*...*/}

// RootLayer.cs
this.Add("root-layer")
	.WithDragToPickBehavior(Store.Instance.DraggedItem, Store.Bus)
	.WithGhostItemBehavior<BasicItemView>(Store.Instance.DraggedItem)
	.WithItemTooltipBehavior<Tooltip>();

```
