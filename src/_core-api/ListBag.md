---
order: 990
title: ListBag 🚧
---


### Events

#### ItemChanged
```cs
public event Action<ListSlot> ItemChanged;
```

#### CollectionChanged
```cs
public event Action CollectionChanged;
```

#### CollectionReset
```cs
public event Action CollectionReset;
```





<!-- 
public string Name;

public virtual IEnumerable<Item> Items { get => Enumerable.Empty<Item>(); }

public virtual Result CanAdd(Item item) => Result.Success;

public virtual Result CanRemove(Item item) => Result.Success;



public virtual Result AddRange(IEnumerable<Item> items) => Result.Fail;

public virtual Result Remove(Item item) => Result.Fail;

public virtual Result TransferAll(Item fromItem, Slot toSlot, Item toItem) => Result.Fail;

public virtual Result TransferOne(Item fromItem, Slot toSlot, Item toItem) => Result.Fail;

public virtual Result SplitHalf(Item item) => Result.Fail;



public virtual void Reset() { }

public virtual void Init() { }

public virtual void Clear() { }

public virtual bool Accepts(Item item) => true;

public virtual bool AllowStacking() => true;

public virtual Slot FindSlot(Item item) => null; -->