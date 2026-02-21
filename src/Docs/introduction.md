---
order: 1010
title: Introduction
---

**Inventory Framework** draws inspiration from games like **Minecraft**, **Dark Souls**, **Witcher 3**, **Diablo II**, **Path of Exile**, **Backpack Battles**, **DayZ**, and **Escape from Tarkov**.

The demos included in the package emulate behaviors from these games to a degree. The goal being to demonstrate the flexibility of the underlying architecture.

### Features and mechanics

The framework covers typical features one might expect from an inventory system, while avoiding rigid assumptions about how your game should behave.

Core behaviors include 
- **drag-and-drop**, 
- **item stacking and splitting**, 
- **tooltips**, 
- **item rotation** (for grid-based inventories), 
- and **slot restriction** based on item type. 

These mechanics work out of the box and are designed to be composed and extended.

Other common mechanics such as 
- **dropping items and spawning prefabs**, 
- **equipping items**,
- **buying and selling**, 
- **crafting**, 
- **container items** (items that hold other items), 

are not part of the core domain and instead are provided as **example/demo implementations** meant to be studied, extended and adapted.

The framework supports both **list-based** inventories, where each item occupies a single cell (Minecraft), and **grid-based** inventories, where items can have rectangular or irregular shapes. This enables game-defining mechanics such as **"inventory Tetris"** (Backpack Battles, Diablo). 

Features and mechanics like these are deceptively complex. Supporting rotation, collision checks, spatial validation and visual feedback typically requires a significant engineering effort. The framework abstracts much of this complexity away.

![Inventory Tetris](/static/images/inventory-tetris.jpg)

### UI Toolkit

**UI Toolkit** is the UI system of choice (*for now*). It is well-suited for inventory interfaces, due to it's declarative nature and built-in flexible layout system. 

**UI Builder** allows you to iterate on styling and layout without recompilation, which significantly improves workflow when designing inventory screens. **Styling** through **themes** and **USS** rules can be extremely powerful, though it sometimes introduces its own challenges when managing specificity and debugging style conflicts. 

**Transitions** (scale, rotation), **hover states** (border on mouse over), and **runtime theme switching** come "*for free*" in **UI Toolkit** as opposed to traditional `GameObjects` and `Prefabs`.

That being said, **UGUI** is still a *thing* and support for it is on the [roadmap](/docs/roadmap)!

![UI Builder](/static/images/ui-builder.jpg)