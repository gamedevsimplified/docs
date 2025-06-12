---
order: "700"
---

### How to use the demos

The demos serve both as a showcase of what's possible and as a solid starting point for a custom inventory system.  
They are designed to be copied into projects and modified as needed.  
You can start with the `Minimal` version and add the features you need, or with `Basic` (I might rename this to `Minecraft` if the Asset Store allows it) and trim what you don’t need while adding your own features. When copying the whole thing, refactor/rename the namespaces in all files to something that makes sense to you. Some demos may include files that have the demo name in their names. You should rename those too, for consistency. Some editors may help you with automating this but I don't even bother and do it by hand - there aren't *that* many files.

### How do you decide what you need?

### The Store

First, there’s the `Store`, which is the central place where data is stored and handled.  
The `Store` must listen to at least two events—`pick item` and `place item`.  
Handlers for these events pass the data to `Core` functions for processing. Check either demo for examples of this.  
For instance, in `Basic`, moving an item between inventories (such as with `ctrl+click`), is handled in the `on pick` event handler.

The `Store` must also track the currently dragged item, which is an `Observable<Item>`.  
Why an observable? So that different parts of the system can react to changes without needing to explicitly subscribe to a `Dragged Item Changed` event.  
The `Store` is a singleton, and that’s fine—everything in it is read-only.

### The Type System and Items

You will need to enhance the type system to fit to your needs.
The `Minimal` demo doesn’t expand on the type system and directly creates items in the `Store`. 
However, `Basic` introduces additional base types and adds `Rarity` to item data (see `Basic/Inventory/Types.cs`).

All items have `quantity` and `stackable` fields—these seemed like the only universally shared properties across all inventory types.  
Another common property is the `icon path`, which is also included in base item data.

Why are items split into `Base` and `Data`?  
This follows the [Flyweight pattern](https://en.wikipedia.org/wiki/Flyweight_pattern), an efficient design pattern that embraces composition over inheritance.  
For example, an `Apple` instance doesn’t need to carry redundant, unchanging data like its icon or description—it can reference a shared base instead.

### The UI System

You will need a `UI Document` attached to a `GameObject` in your scene, a `Panel Settings` asset, a `Root UXML` asset, and a script to attach the inventory’s root to the document.

You can start with the built-in renderers for items, slots, and "bags" (inventories) and create custom views as needed.  
For example, in the `Basic` demo, the custom `BasicItemView` is a copy of `ItemView` with an extra background element to reflect item rarity. You could argue that it should extend the base instead, but for components this small, it's not worth the trouble. More on this later.

You're expected to create custom views for your inventory screens and windows, using a mix of built-in and custom views alongside other UI elements like buttons and text.  
For an example, see `Basic/Views/ChestWindow.cs`. It uses the built-in `ListBag` renderer and adds a button to collect all items.

The UI should mainly define structure and styling, with minimal behavior (such as reacting to button clicks).  
Core logic is abstracted into extension methods (see the next section for details).

Each demo includes a `Theme` file (see `Resources/Basic/BasicTheme.tss`).  
Technically, you could use the default theme, but it's quite ugly, and you’ll likely customize it anyway.

Most styling is consolidated into a single large USS file (`BasicStyling.uss`).  
Although not the cleanest approach, I found it easier to search for selectors in one big file rather than looking in many smaller ones.  
If some styles are unrelated to the rest, you can move them into separate files and import them into the theme (see `BasicTooltip.uss`).  
This is just a suggestion—organize your styles however makes the most sense to you.

If something looks off, a missing or incorrect style is likely the cause.  
To help with debugging, each demo includes a window that displays the raw state of the `Store`.  
To open it, go to `Tools > Basic > Debug`. You’re expected to modify this window (through code, of course, see `Basic/Editor/Debug.cs`) and add any extra data you need to observe.

### Behaviors and Interactions

You’ll need to attach behaviors to the root layer of your UI document (see `Basic/Views/RootLayer.cs`).  
These behaviors handle visual interactions, such as showing the dragged item or displaying tooltips on hover (see `Core/Behaviors/Behaviors.cs`) .

For example, `WithRestrictedSlotBehavior` observes the **dragged item** and reacts to _mouse over/out_ events on all `SlotViews`.  It checks whether the slot accepts the dragged item (defined when the slot is created) and visually marks the slot as legal or illegal by adding a USS class (`"legal-action"`).  
This applies a red or green overlay on the slot (assuming the relevant styles are defined).

In order to be able to pick (and place) items and see them move you will need to add a `pick` and a `ghost item` behavior. Currently there are two `pick` behaviors and one `ghost` behavior defined (placing is also defined in the pick behavior):
- `WithClickToPickBehavior` will pick an item on mouse down
- `WithDragToPickBehavior` will pick an item when it was dragged a minimum amount of pixels (`32` by default), as well as on mouse up
- `WithGhostItemBehavior` watches mouse for movement and renders a view of the dragged item.