---
order: 900
title: Architecture Overview
---

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
public class Basic_ItemBase : ItemBase {
	public int Weight = 1;
    public int Cost = 1;
    public override Item CreateItem() => new Basic_Item {/*...*/}  
}

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

```cs Tag.cs
public class Tag : ScriptableObject { }
```

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

The **Store** is the central authority responsible for enforcing business rules, updating state (**Bags**), and publishing result events. It coordinates user input, inventory state, and external systems, acting as the single source of truth for state transitions.

A **Store** is defined as a **ScriptableObject** and can contain globally accessible state, acting as a singleton.

Rules defined in the **Store** operate on the system level - they take the entire system state into account rather than operating on isolated components. For example, the **Store** can define what should happen when a player `Ctrl+clicks` an item while the shop window is open, determine whether an item can be stacked or split, or coordinate interactions between multiple bags.

By convention, only the **Store** is allowed to modify **Bags**. This ensures predictable state transitions, centralized validation, and a clear architectural boundary. **Bags** themselves are passive data structures; they do not enforce global or cross-bag rules.

When state changes occur, the **Store** publishes result events through an **Event Bus**. This allows other systems to react without maintaining hard references to the source of the change. For example, a **MonoBehaviour** can play a sound when an item is successfully picked up or placed, UI components can animate slot updates, or game systems can respond when an item is dropped into the world.

The abstract **Store** contains a reference to an Event Bus and a Ghost Item. The Ghost Item represents the item currently being manipulated, such as during drag-and-drop operations.

For simple scenarios, **Stores** defined in **Examples** can be reused. The **Minimal Example Store** supports basic drag-and-drop behavior, while the **Stacking Example Store** extends this with stack splitting and stacking rules. More advanced use cases require custom Store implementations. For example, the **Basic Demo Store** creates a bridge between the UI and the game world by defining rules for dropping items from the inventory into the world.

<hr>

### Event Bus

The **event bus** allows systems to communicate without direct references. By convention, the **event bus** is defined within an abstract **store** and is injected into any component that needs to publish or subscribe to events.

When a component publishes an event, it follows a “fire-and-forget” approach, meaning it does not wait for a response. This reflects the intent behind the event rather than performing the actual state mutation. The responsibility for modifying the system’s state lies elsewhere, typically within the **store**.

Events typically fall into two categories: **intent events** and **result events**. **Intent events** describe an action that should be performed, such as picking or placing an item, without guaranteeing that the action will succeed. **Result events**, on the other hand, communicate the outcome of that action, such as success or failure. 

For example, the `DragDropManipulator` publishes **Pick** and **Place** events when the user interacts with an inventory slot. The **store** listens to these events and updates the underlying data (**Bags**) accordingly. After processing, it emits either a **Success** or **Fail** event on the same bus, depending on the outcome. Other systems, such as an SFX manager, can subscribe to these result events and react independently, for instance by playing different sounds for success or failure cases.

```cs DragDropManipulator.cs
bus.Publish(new PickItem(context.Bag, context.Slot, context.Item, e));
```

This approach makes components optional and encourages a modular, extensible design. Systems can be added or removed without introducing tight dependencies or risking unintended side effects across the application.

The event bus should primarily be used for communication between independent systems or components that should remain loosely coupled. For localized or tightly scoped interactions, direct method calls are often more appropriate. 

!!!info 
One trade-off of this pattern is that it can be more difficult to trace the flow of events throughout the system. To mitigate this, all events are logged to the console by default, making it easier to observe and debug the event-driven interactions.
!!!

<hr>

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

<hr>

### Controller

Controllers do not define gameplay rules. They simply connect the parts of the system together.

They:

- Create or reference Bags
- Inject Bags into Stores
- Bind Bags to Views
- Attach behavior in form of Manipulators

