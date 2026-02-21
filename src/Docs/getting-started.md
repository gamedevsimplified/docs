---
order: 1000
title: Getting Started
---

The best way to get started is by exploring the demos and examples. 

Examples are isolated implementations of various use-cases. The idea is to separate the solution to a problem from all the potential noise. Examples are broken down by complexity into **basic**, **intermediate**, **advanced** (and **grid**, in `PRO` version), ranging from the **simplest list inventory** to **container items** and **crafting benches**. More examples will be added over time.

![Examples List](/static/images/examples-list.jpg)


### Example structure

A typical example consists of a: 
- Scene
- UI Document
- Controller
- Store

![Minimal Example](/static/images/minimal-example.jpg)

Files are prefixed with the example name to prevent naming conflicts and to keep the examples self-container. This structure also makes it easier to copy an example into your own project and adapt it incrementally.

Some examples introduce custom behavior by extending the **Store**. For instance, splitting a stack or moving an item, are typically implemented inside a custom, reusable **Store**. You can read more about this in [Architecture Overview](/docs/architecture-overview).

### How to use

The recommended approach is to build your functionality by composing and extending existing pieces rather than modifying the framework code directly. Avoid editing demo code in place as updates to the package may overwrite your changes. Instead, treat the provided implementations as references and starting points.

Start by scaffolding your item system. Identify what is fixed (shared properties) and what differentiates items (stats, behaviors, categories). Create new **Items** and **ItemBases** for each meaningful item type. This allows you to define rule-based logic, like showing different tooltips for weapons versus armor, or preventing certain item types from being placed into specific equipment slots.

Next, identify which type of inventories your game should support - player inventory, stash, equipment, shop, crafting interface, etc. Create distinct types for each sufficiently different screen. This separation enables rule-driven behavior based on context: where an item originates, where it is placed, and what actions are permitted between those contexts.

For basic interactions, reuse an existing Store implementation. When your design requires new rules or workflows, create a custom Store derived from an example.

Introduce new Event types when necessary. For example, the Basic demo adds a custom event triggered after a successful crafting operation. That event is then consumed by a separate SFX controller to play a "crafting" audio clip. 

Finally, create your **UI Document** and **Controller**, wire them together with your **Store**, and start with simple interactions between inventories. Add UI toggles, feedback, and additional views gradually. Iterate.