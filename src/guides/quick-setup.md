---
title: Quick Setup
order: 1000
---

!!!
This setup uses the `Basic` template as an example.
!!!

https://www.youtube.com/watch?v=RS3KWIsWBCY


To quickly add an inventory to your scene, locate the `Inventory` prefab in one of the demo/example `Prefabs` folders and drag it into your scene.

<!-- TODO: explain what the prefab contains -->

### 1. Add the *Inventory* prefab

Navigate to the desired demo folder (`GDS/Demos/Basic` in this example), and drag the `Inventory` prefab from the `Prefabs` folder into your scene hierarchy.

![](/static/images/tutorials/quick-setup-1.jpg)

---
### 2. Link the *Player Transform*

If you don't already have a `Player` object in your scene, create a **Cube** and name it `Player`.

Select the `Inventory` object in the scene, expand the `InventoryEvents` component, and drag the `Player` object from the hierarchy into the `Player Transform` field.

**Items** dropped on the ground will spawn at random positions around the **Player**.


![](/static/images/tutorials/quick-setup-2.jpg)

---
### 3. Add an *Event System*

If you don't already have an `Event System` in your scene, create one by going to `GameObject > UI > Event System`.

An **Event System** is required in the scene because it acts as the central hub that processes input from the new **Input System** and dispatches it to UI elements or game objects.

![](/static/images/tutorials/quick-setup-3.jpg)

---
### 3. Add a *Physics Raycaster* to Main Camera

Select the `Main Camera` in *Hierarchy* and add a `Physics Raycaster` component.

The **Physics Raycaster** is needed on the main camera so that the new **Input System**’s UI and **event system** can detect and interact with 3D objects through raycasts.

![](/static/images/tutorials/quick-setup-4.jpg)
