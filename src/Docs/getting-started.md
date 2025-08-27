---
order: "1000"
---

## Introduction

The purpose of this package is to provide a versatile and extensible solution for creating inventories in Unity. Its design allows developers to quickly set up functional inventories while offering the flexibility to adapt to custom use cases. The framework supports a variety of inventory types, making it suitable for a wide range of games. 

The predefined inventory types are:
   - **List**: A straightforward collection of items arranged as a list or a grid.
   - **Equipment**: Ideal for RPGs or games with wearable gear.
   - **Grid** (PRO): Supports items of varying sizes.

Built on [UI Toolkit](https://docs.unity3d.com/Manual/UIElements.html), it offers advanced styling and layout capabilities while avoiding the overhead of `GameObjects` and `Prefabs`. 
It also comes bundled with an abstract yet powerful **item system** that minimizes setup while supporting diverse use cases.

This framework is currently targeted at **programmers**. It lacks designer-friendly features such as custom editors or Scriptable Objects, so some coding is required to implement custom functionality. 

## Getting started

To get started, check out the 2 demos included - "Minimal" and "Basic".

The **Minimal** demo has the least amount of code needed to setup a working inventory. It is a great starting point to familiarize yourself with the fundamentals of the framework. It consists of one inventory, some items and the simplest `behavior` of moving items around.

![](/static/images/minimal.png)


The **Basic** demo is a *Minecraft*-inspired inventory featuring the following:
- A `main inventory` window consisting of equipment, inventory and a consumables hot-bar
- A `shop` window where you can buy pre-defined items and sell items from your inventory.
- A `treasure chest` window, which has some items that you can pick but cannot place anything back. You can also pick all at once.
- A `crafting bench` window, which has slots for materials and an outcome slot. Placing materials in the correct order will populate the outcome slot. Picking the item from outcome slot will "craft" the item.
- A `stash` window, which is an inventory you can store items in.
- A character sheet window
- A way to drop items on the "ground". 
- A way to destroy items. 
- All this is integrated in a minimalistic 3d scene.

![](/static/images/basic.png)


## Installation

You can install the package from Unity Package Manager.

The package has one dependency - the **New Input System**.

[!ref](/guides/new-input-system.md)