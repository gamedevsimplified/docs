---
order: 750
title: Core API 🚧
---

{{ include "snippets/wip" }}

## Bag

> use `BagExt` static class if function is not an extension method

### AddItems

Adds Items to a Bag.

```cs
public static Result AddItems(this Bag bag, params Item[] items);
```

### PickItem

Picks an Item from a Bag.

```cs
public static Result PickItem(PickItem e);
public static Result PickItem(this Bag bag, Slot slot, EventModifiers mods = EventModifiers.None);
```

### PlaceItem

Places an Item in a Bag.

```cs
public static Result PlaceItem(PlaceItem e);
public static Result PlaceItem(this Bag bag, Slot slot, Item item, EventModifiers mods = EventModifiers.None);
```

### MoveItem

Moves Items from a Bag to another target Bag.

```cs
public static Result MoveItem(PickItem e, Bag toBag);
public static Result MoveItem(this Bag bag, Slot slot, Bag toBag);
```

### PickOrMoveItem

Picks an item from the source bag, or moves it to the target bag if Ctrl is held.

Returns a failed result if moving but no target bag is provided.

```cs
public static Result PickOrMoveItem(PickItem e, Bag toBag)
```

---
## ItemBase

### Create

Creates an `Item` from an `ItemBase`

```cs
public static Item Create(this ItemBase itemBase)
```

---
## Item

### Clone
Clones an Item and returns it.
```cs
public static Item Clone(this Item item)
```

### Size
Returns Item Size if it's a ShapeItem or a 1x1 
```cs
public static Size Size(this Item item)
```

### SetQuant
Creates a new Item with the specified quantity.
```cs
public static Item SetQuant(this Item item, int quant)
```
### Decrement
Creates a new item with quantity decreased by 1.
```cs
public static Item Decrement(this Item item)
```
### StackAll
Moves all quantity from source item to target item.
```cs
public static (Item from, Item to) StackAll(Item fromItem, Item toItem)
```
### StackOne
Decrements source Item quantity and increments target Item quantity.
```cs
public static (Item from, Item to) StackOne(Item fromItem, Item toItem)
```
### UnstackHalf
Splits a stack in half.
```cs
public static (Item from, Item to) UnstackHalf(Item fromItem)
```
### CanStack
Checks if an Item can be stacked onto antoerh Item.
```cs
public static bool CanStack(this Item fromItem, Item toItem)
```

---
## Slot

### IsEmpty

Returns true if Slot is empty.
```cs
public static bool IsEmpty(this Slot slot)
```
### IsFull

Returns true if Slot is not empty.
```cs
public static bool IsFull(this Slot slot)
```
### Clear

Returns a new empty Slot.
```cs
public static T Clear<T>(this T slot) where T : Slot
```


