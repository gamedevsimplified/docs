---
title: Quick Setup
order: 1000
---
!!!
This setup uses the `Basic` template as an example.
!!!

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
