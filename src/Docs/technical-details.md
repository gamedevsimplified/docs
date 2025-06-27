---
order: "800"
title: Technical Details
---

## Functional programming

The framework takes a **functional programming approach**, emphasizing simplicity and separation of concerns. It uses the a common "good" practice of *Composition* over *Inheritance* and not so common convention of *Rules* over *Conditions* (pattern matching).
```cs
public static Result PlaceItem(this Bag bag, Item item, Slot slot){
	return (bag, slot) switch {
		(GridBag b, GridSlot s) => PlaceItem(b, item, s),
		(SetBag b, SetSlot s) => PlaceItem(b, item, s),
		(ListBag b, ListSlot s) => PlaceItem(b, item, s),
		_ => (false, Item.NoItem)
	};
}
```

## Records

[**C# Records**](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) are used to model data types. They offer concise syntax, value-like behavior and are immutable by default. The [immutability](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#immutability) aspect is particularly useful for **View** Components, as it simplifies determining whether a re-render is necessary.

```cs
public record Item
```

## Extension methods
A type's behavior is implemented via **extension methods** to keep the data structures clean and focused. This prevents behavior inheritance, which is a big pain-point in classic OOP design.

```cs
public static Slot Clear(this Slot slot) => slot with { Item = Item.NoItem };
```

## Factories
To ensure proper input validation when creating new instances, each data type comes with one or more **factory** functions. For example, in `Basic` demo, an **Item** can be created by providing just an `ItemBase`, or `ItemBase` and `Quant`, or `ItemBase`, `Quant` and `Rarity`

```cs
public static class ItemFactory {
	public static BasicItem Create(this ItemBase itemBase, Rarity rarity, int Quant = 0) => itemBase switch {
		ArmorBase b => new ArmorItem(b) { Rarity = rarity },
		WeaponBase b => new WeaponItem(b) { Rarity = rarity },
		_ => new BasicItem() { Base = itemBase, Rarity = rarity, Quant = Quant }
	};
	public static BasicItem Create(this ItemBase itemBase, int Quant) => Create(itemBase, Rarity.NoRarity, Quant);
	public static BasicItem Create(this ItemBase itemBase) => Create(itemBase, Rarity.NoRarity);
}
```

## Null object

To represent the absence of an object the [**null object pattern**](https://en.wikipedia.org/wiki/Null_object_pattern) is employed in favor of the classic *null* reference. This means that most types have an associated void type: `Item => NoItem` which acts like the [identity element](https://en.wikipedia.org/wiki/Identity_element#:~:text=In%20mathematics%2C%20an%20identity%20element,such%20as%20groups%20and%20rings.) from algebra. One of the main advantages of this approach is that an operation can be performed on a an item or a collection of items without the need of a null check beforehand. An operation like this will have no effect on the item, if it is a `NoItem`.

```cs
public record Item {
	public static NoItem NoItem = new();
}
```


## UI Toolkit

The framework leverages **UI Toolkit** for rendering inventory views. It provides a declarative way to define elements, their data, and their styles using C# and USS, while completely avoiding **UXML**. The reason for bypassing UXML is that data binding, referencing elements and working with the  UXML asset, all come with levels of verbosity and complexity that drastically slow down productivity. This means, however, that the included visual components (inventories, windows, slots) cannot be viewed in Unity's **UI Builder**. You can still use the UI Builder to design own custom components if you prefer so.
Since UI Toolkit is heavily based on web technologies, the framework proposes web-inspired nomenclature, such as:
   - [**DOM**](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model): Utility functions for managing elements.
   - [**DIV**](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/div): A shorthand alias for `VisualElement`.


<!-- ## How it works

From a birds eye view, the system consists of a `data store`, an `event bus` and `UI` 
When the user interacts with the UI, let's say *picks an item from a slot*, an event is put on the event bus.
```cs
// Behaviors.cs
CustomEvent evt = new PickItemEvent(t.Bag, t.Data.Item, t.Data, e.modifiers)                    

bus.Publish(evt);
```

The `Store`, having subscribed to the event, sends its data to a handler function which will update the `bag`.
```cs
// Store.cs
Bus.Subscribe<PickItemEvent>(e => OnPickItem(e as PickItemEvent));
void OnPickItem(PickItemEvent e) {
	//...
	bag.Notify();	
}
```

Then, any view that is observing the bag will update itself.
```cs
// ListBagView.cs
this.Observe(bag.Data, (slots) => {/*...*/});
``` -->

