---
order: 900
title: Architecture Overview 🚧
---

{{ include "snippets/wip" }}

The framework follows a **data-driven**, **event-based architecture**. It keeps data, logic, and UI separate.

Inventories can get complex very quickly — stacking, moving, crafting, containers, world drops, sounds, etc.

To keep things manageable and extensible, the system is divided into clear responsibilities.

- **Bags** → hold data
- **Stores** → define rules
- **Views** → render UI
- **Manipulators** → add UI Interaction Behaviors
- **Controllers** → wire everything together
- **Event bus** → connects these pieces without tightly coupling them.

### Bag

Bags are pure data containers. Think of a Bag as the state of an inventory — nothing more.

A Bag stores items, does not contain UI logic but can define some gameplay rules. For example a bag can be configured to be remove only or accept only a certain type of items.

Because Bags are UI-agnostic, they can be reused across different interfaces or even outside UI contexts (e.g., crafting systems, world storage, containers inside items).

Examples: Equipment, Shop, Crafting Bench


### Item

Items are based on ScriptableObject definitions (ItemBase) with runtime instances (Item).

Items describe what something is, not how it behaves in the UI.

This allows:
- Easy editor configuration
- Reusable item definitions
- Custom item types (e.g., container items)

### Store

Stores contain the logic of the system.

A Store:

- Listens to events (e.g., Pick, Place, Drop)
- Applies validation rules
- Updates Bags
- Publishes result events

For example, a Store decides whether an item can be moved, whether stacks can merge, what happens when an item is dropped into the world.

Stores define what is allowed, but they do not render anything. This makes behavior easy to customize — you can swap or extend Stores without changing Views.

### Manipulator

Manipulators are small, reusable interaction components attached to the root visual element. They are responsible for translating user input into high-level intent.

For example, Manipulators enable behaviors such as:
- Drag and drop
- Tooltips
- Item rotation
- Right-click / double-click actions
- Slot highlighting

A Manipulator typically:
- Listens to pointer or keyboard events.
- Applies activation rules (e.g., modifier keys, mouse buttons).
- Publishes an event to the event bus.

Importantly, Manipulators do not modify Bags directly. They only interpret input and emit intent.

### Controller

Controllers do not define gameplay rules. They simply connect the parts of the system together.

They:

- Create or reference Bags
- Inject Bags into Stores
- Bind Bags to Views
- Attach behavior in form of Manipulators

### Event Bus

The event bus allows systems to communicate without direct references.

For example:

- A Manipulator publishes a "Pick" event.
- The Store reacts and updates a Bag.
- The View re-renders automatically.
- A sound system listens and plays an effect.

Because everything communicates through events:
- Features can be added without modifying core systems.
- Sound, VFX, and world spawning remain optional.
- The framework stays modular and extensible.



<!-- The framework is built around several key concepts that define its structure and behavior. Understanding these will help you make the most of its features.

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

``` -->
