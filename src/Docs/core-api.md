---
order: 750
title: Core API 🚧
---

{{ include "snippets/wip" }}

### Bag (Base Class)

Bag is the abstract base class for all inventory containers.

It defines the common contract for storing, validating, and manipulating Item instances.

Concrete implementations (e.g., ListBag, GridBag, CraftingBench) provide the actual behavior.

The base implementation intentionally does very little — most methods return Result.Fail by default.

Derived classes are expected to override the relevant methods.

<!-- 
public string Name;

public virtual IEnumerable<Item> Items { get => Enumerable.Empty<Item>(); }

public virtual Result CanAdd(Item item) => Result.Success;

public virtual Result CanRemove(Item item) => Result.Success;

public virtual Result Add(Item item) => Result.Fail;

public virtual Result AddAt(Slot slot, Item item) => Result.Fail;

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