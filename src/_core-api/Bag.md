---
order: 1000
title: Bag 🚧
---

{{ include "snippets/wip" }}

Bag is the abstract base class for all inventory containers. It defines the common contract for storing, validating, and manipulating Item instances. The base implementation intentionally does very little — most methods return Fail or Success by default. Concrete implementations (e.g., ListBag, GridBag) provide the actual behavior. Derived classes are expected to override the relevant methods.

### CanAdd
```cs
public Result CanAdd(Item item);
```

### CanRemove
```cs
public Result CanRemove(Item item);
```


### Add
```cs
public Result Add(Item item)
```

### AddAt
```cs
public Result AddAt(Slot slot, Item item);
```

### AddRange
```cs
public Result AddRange(IEnumerable<Item> items);
```

### Remove
```cs
public Result Remove(Item item);
```

### TransferAll
```cs
public Result TransferAll(Item fromItem, Slot toSlot, Item toItem);
```

### TransferOne
```cs
public Result TransferOne(Item fromItem, Slot toSlot, Item toItem);
```

### SplitHalf
```cs
public Result SplitHalf(Item item);
```

<!-- 
public string Name;

public virtual IEnumerable<Item> Items { get => Enumerable.Empty<Item>(); }








public virtual void Reset() { }

public virtual void Init() { }

public virtual void Clear() { }

public virtual bool Accepts(Item item) => true;

public virtual bool AllowStacking() => true;

public virtual Slot FindSlot(Item item) => null; -->