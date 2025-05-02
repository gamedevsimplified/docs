---
order: "900"
---

The framework is built around several key concepts that define its structure and behavior. Understanding these will help you make the most of its features.

### Item

An **Item** is a data type that represents an individual object in an inventory. It is highly flexible and comprises two main components:
- **Base**: Contains fixed properties like ID, name, and icon path.
- **Data**: Holds dynamic properties such as quantity or rarity.

```cs Item.cs
public record Item(int Id, ItemBase ItemBase, ItemData ItemData)
public record ItemBase(string Id, string Name, string IconPath, bool Stackable, Size Size);
public record ItemData(int Quant = 1);
```

This architecture is heavily inspired from RPGs like [Diablo](https://diablo2.io/base/) and [Path of Exile](https://www.poewiki.net/wiki/Body_armour) but the separation ensures adaptability to fit various gameplay scenarios. 


### Slot

A **Slot** is a container that can either hold an **Item** or remain empty. Slots can enforce **restrictions**, such as allowing only specific types of items, by providing a function (`Accepts`) that operates on `Item` type. 

```cs
// Inventory.cs
public delegate bool FilterFn(Item item);
public record Slot(Item Item) { 
	public FilterFn Accepts = (_) => true; 
};
```

The base `Slot` type is extended into specialized slot types: 
- `ListSlot`: Index-based storage.
- `SetSlot`: Named slots for equipment/wearable items.
- `GridSlot` (_not available in Lite_): Grid-based position storage.


### Bag

The **Bag** represents an inventory. It manages a collection of **Slots**. Each inventory type comes with its own behavior and can enforce restrictions on accepted items. 

```cs
// Types.cs
public abstract record Bag(string Id) {
	public static NoBag NoBag = new();
	public FilterFn Accepts = Filters.Everything;
}
```

The base `Bag` type is extended into specialized types:
- `ListBag`: Contains a list of `ListSlots`
- `SetBag`: Contains a list of `SetSlots`
- `GridBag` (_PRO_): Contains a 2D array of `GridSlots` as well as a list of `GridItems`


### Event Bus and Events

The framework uses an [**Event Bus**](https://dzone.com/articles/design-patterns-event-bus) to decouple its components. **Events** are data containers describing actions, such as picking up an item. For example, a `PickItem` event includes the item itself, the source inventory and slot.

```cs
// Events.cs
public record PickItemEvent(Bag Bag, Item Item, Slot Slot);
```

An event does not perform any actions itself, that falls under the responsibility of the **Store**.

### Store

Inspired by web technologies like [**Redux**](https://redux.js.org/faq/general#when-should-i-use-redux), the **Store** serves as the central hub for managing system state. It subscribes to events on event channels and performs necessary state updates. All changes to the system state flow through the **Store**, ensuring consistency and predictability.

```cs Store.cs
public Store() {
	Bus.Subscribe<PickItemEvent>(e => OnPickItem(e as PickItemEvent));
	// ...
}

void OnPickItem(PickItemEvent e) {
	LogEvent(e);
	// ...
}
```


### Item Bases

This is your repository of item bases and all the types that fully describe and categorize them.
For example an *Apple* is defined as a **stackable**, **consumable** **basic item**, while a *Dagger* is a **1-handed** **weapon**, with *Attack Damage* and *Attack Speed*

```cs Bases.cs
public static ItemBase Apple = new("Apple", "Apple", "Shared/Images/items/apple", true, ItemClass.Consumable);
public static WeaponItemBase ShortSword = new("ShortSword", "Short Sword", "Shared/Images/items/sword-blue", ItemClass.Weapon1H, 80, 1.5f);
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
