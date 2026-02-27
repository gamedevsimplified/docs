---
order: 900
title: Architecture Overview 🚧
---

{{ include "snippets/wip" }}

### Overview

The framework follows a **data-driven**, **event-based** architecture. It keeps the data, logic and UI separate.

Here's a bottom-up bird's-eye view of the architecture:
- **Items** are defined as `Scriptable Objects`.
- **Items** are stored in collections called **Bags** (List, Set, Grid). 
- **Bags** are collections of slots with rules defining what items can be stored and how. 
- **Bags** are typically defined in **Controllers**. 
- **Controllers** are `MonoBehaviors`. 
- **Controllers** query the [**UI Document**](https://docs.unity3d.com/6000.3/Documentation/Manual/UIE-create-ui-document-component.html) for **Bag Views**, initialize them and attach [**Manipulators**](https://docs.unity3d.com/6000.3/Documentation/Manual/UIE-manipulators.html).
- **Manipulators** implement behaviors (drag-and-drop, tooltip). 
- **Manipulators** react to user input and publish events on the **Event Bus**. 
- The **Event Bus** is a one-way messaging mechanism that decouples  UI from logic and data. 
- The **Event Bus** is declared in the **Store**. 
- The **Store** is a `Scriptable Object`.
- The **Store** is the sole authority over data mutation. 
- The **Store** behaves like a Singleton (it *was* a **Singleton** in **v1**). 
- The **Store** processes events and updates the **Bags** (either passed as event parameters or from its own context). 
- **Bags** notify any *subscribers* of data change (typically **Bag Views**). 
- **Bag Views** update themselves in response to data changes. 
- **Scripts** other than the **Store**, can listen to events on the **Event Bus** and trigger their own behavior (play sounds, spawn objects).

The system follows a unidirectional flow: user input generates events, the Store processes them, state updates, and views react to changes:

```
UI → Manipulator → EventBus → Store → Bag → BagView
```

To sum up here are the major components and their responsibilities:
- **Bags** → hold data
- **Stores** → define rules and mutate data
- **Views** → render UI
- **Manipulators** → add UI Interaction Behaviors
- **Controllers** → wire everything together


<!-- The framework follows a **data-driven**, **event-based architecture**. It keeps data, logic, and UI separate.

Inventories can get complex very quickly — stacking, moving, crafting, containers, world drops, sounds, etc.

To keep things manageable and extensible, the system is divided into clear responsibilities.

- **Bags** → hold data
- **Stores** → define rules
- **Views** → render UI
- **Manipulators** → add UI Interaction Behaviors
- **Controllers** → wire everything together
- **Event bus** → connects these pieces without tightly coupling them. -->

<hr>

### Item System

Item systems in games vary significantly in scope and complexity. Some games feature complex systems with multiple item categories (weapons, armor, consumables), subtypes, rarity tiers, affixes, requirements, etc. Other games use colored shapes and nothing else.

To allow such variety, the item system was designed to be as **abstract** and as **extensible** as possible.

At a minimum, an item defines:
- an icon
- whether it is stackable
- maximum stack size 
- current stack size 

Items in games typically consist of a **fixed** part and a **dynamic** part. For example, in *Path of Exile* a body armor has a [base](https://www.poewiki.net/wiki/War_Plate) that defines immutable properties (icon, armor range, attribute requirements), while instance-specific values include the actual armor roll, rarity and affixes. In contrast, in *Backpack Battles*, [items](https://backpackbattles.wiki.gg/wiki/Leather_Armor) are entirely static except for their orientation within the inventory.

This structural separation is known as the [Flyweight pattern](https://www.geeksforgeeks.org/system-design/flyweight-design-pattern/): shared immutable data is stored once, while instance-specific state is stored separately.

In our case, the fixed part of the **Item** is a type called **ItemBase**, which consist of the icon, the stackable flag and maximum stack size. The **Item** itself stores the current stack size.

!!!question What if you're already using another item system?
You can bridge an existing item system with Inventory Framework. A guide for that will be added soon.
In the meantime, reach out on Discord.
!!!

<hr>

### ItemBase

**ItemBase** is implemented as a **ScriptableObject**. It describes what an item is, and not how it bahaves.

Each **ItemBase** has a factory method responsible for creating and initializing an **Item** instance of the associated type.

For example, in the **Basic Demo**, the Axe **ItemBase** has a fixed attack damage range. Upon instantiation, the Axe instance will roll attack damage from its base attack range and store in `AttackDamage` property.

```cs
public class Basic_WeaponBase : Basic_ItemBase { /* ... */	
	public IntRange DamageRange;
	public override Item CreateItem() => new Basic_Weapon { /* ... */		
		AttackDamage = DamageRange.Roll() 
	};
}
```

To extend the functionality of an **Item**, extend the **ItemBase** and override the factory method.
For example, all items in the Basic demo have a fixed `Weight` and `Cost`, but variable `Rarity`:

```cs
    [CreateAssetMenu(menuName = "SO/Demos/Basic/Basic_ItemBase")]
    public class Basic_ItemBase : ItemBase {
        public int Weight = 1;
        public int Cost = 1;
        public override Item CreateItem() => new Basic_Item {/*...*/}  

    [System.Serializable]
    public class Basic_Item : Item {
        public Rarity Rarity;
    }
```

!!!info Annotate your Item types with System.Serializable.
This ensures they are visible in the Unity Editor and persist correctly between scene reloads.
!!!

Create as granular a hierarchy as your game requires. A useful rule of thumb: if certain properties only apply to a subset of items, consider introducing a more specific type.

Example hierarchy from the **Basic Demo**:

- Item 
	- BasicItem (weight, cost, rarity)
		- BasicWeapon (damage)
		- BasicArmor (defense)

<hr>

### Tag

**Tag** is **ScriptableObject** marker type (it contains no properties). Tags enable flexible constraints, such as restricting a **Slot** or a **Bag** to specific item types. This is achieved by intersecting tag sets. They are defined as `List<Tag>` on **ItemBase**, **Slot** and **Bag** types.

<hr>

### Bag

A **Bag** is pure **data container**. It represents a well-defined slice of your game state and should be treated as the **single source of truth** for that slice.

**Bag** is an abstract class that defines the contract and shared behavior common to all bags, without enforcing any specific storage model. All state-changing behavior is implemented in concrete subclasses.

The three primary implementations are **ListBag**, **SetBag** and **GridBag**. 

**Bags** expose public methods to **add**, **remove**, or **modify items**. By convention, bag state should be mutated from a single place (for example, the **Store**, or main controller). This prevents hidden side effects and keeps state transitions explicit and predictable. 

**Bags** emit **change events** when their state mutates. Other systems, such as UI views, gameplay systems, or persitence layers subscribe to these events and react accordingly (re-render or recompute internal state). For example, **ListBagView** listens to **ItemChanged** event on a ListBag. When raised, it re-renders only the affected slot. This keeps rendering logic decoupled from state mutation and avoids unnecessary full refreshes.

!!!info 
A Bag can be declared in a MonoBehaviour (state resets when exiting Play Mode), or in a ScriptableObject (state persists after Play Mode and can be reset manually).
!!!

#### ListBag
**ListBag** behaves like a fixed-size list. It contains an ordered collection of slots, indexed by position. In a **ListBag**, item size and spatial footprint is irrelevant. 

**ListBag** is typically used when:
- order matters (hotbar, quick access list)
- layout is purely visual (can be displayed horizontally, vertically or as a grid)
- all items occupy a single logical slot (1x1)

#### SetBag
**SetBag** behaves conceptually like a dictionary. It maps predefined slot keys to item content.

- Slots are identified by semantic keys (*Helmet, Weapon, Ring*). 
- Slot identity is more important than position
- A Slot typically enforces constraints on what it can accept. 

**SetBag** is typically used in Equipment systems.

#### GridBag

**GridBag** models a two-dimensianal spatial container. Unlike **ListBag**, layout is not purely representational. Spatial structure becomes part of gameplay, allowing players to optimize positioning and capacity.

- Items occupy multiple cells.
- Placement requires spatial validation.
- Position and orientation matter.



<hr>

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
