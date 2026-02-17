---
title: Manual Setup (Recommended)
order: 900
---

!!!
This setup uses the `Basic` template as an example. If you're using a cloned template, apply the same steps to your version instead.
!!!

You can manually integrate a demo template into your scene by following these steps:

### 1. Add a _UI Document_

To add a **UI Document**, right-click the hierarchy and select `UI Toolkit > UI Document`.

<!-- !!!
A UI Document is required to render UI Toolkit layouts (UXML and USS) in the scene.
!!! -->

![](/static/images/tutorials/manual-setup-1.jpg)

---
### 2. Assign UI Assets

Select the **UI Document** object in the scene. Navigate to the desired demo folder (`GDS/Demos/Basic` in this example), and drag the **Panel Settings** and **Visual Tree Asset** into their corresponding fields in the **UI Document** component.

These assets handle styling and layout.

![](/static/images/tutorials/manual-setup-2.jpg)

---
### 3. Add Inventory Scripts

Go to `Demos/Basic/Scene/Scripts` and add the `InventoryEvents` and `InventoryInput` scripts to the **UI Document** object.

`InventoryEvents` handles item spawning when dropped and cleanup when picked up.

`InventoryInput` listens for keyboard input and forwards it to the inventory.

![](/static/images/tutorials/manual-setup-3.jpg)

---
### 4. Add a *Player Input* component

With **UI Document** object selected in Hierarchy, add a `Player Input` component.

Drag the **Basic** Input Action Asset into the **Actions** field of the **Player Input** component.

Make sure that the **Default map** is set to `Inventory` and the **Behavior** is set to `Send Messages`.

![](/static/images/tutorials/manual-setup-5.jpg)

---
### 5. Assign the Player Transform

If you don't already have a `Player` object in the scene tree, create a cube and name it `Player`.

Select the **UI Document**, expand the `InventoryEvents` component and drag the `Player` object from the scene into the **Player Transform** field.

![](/static/images/tutorials/manual-setup-4.jpg)

---
### 6. Assign Item and VFX Prefabs

Go to `Common/Prefabs` and drag `WorldItem` prefab into the **Item Prefab** field and `PickupVFX` prefab into the **PickupVFX** field.

**Items** dropped on the ground will spawn at random positions around the **Player**.

You can adjust the **Drop Radius** and **Drop Offset** here.

---
### 7. Add an *Event System*

If you don't already have an `Event System` in your scene, create one by going to `GameObject > UI > Event System`.

An **Event System** is required in the scene because it acts as the central hub that processes input from the new **Input System** and dispatches it to UI elements or game objects.

![](/static/images/tutorials/quick-setup-3.jpg)

---
### 8. Add a *Physics Raycaster* to Main Camera

Select the `Main Camera` in *Hierarchy* and add a `Physics Raycaster` component.

The **Physics Raycaster** is needed on the main camera so that the new **Input System**’s UI and **event system** can detect and interact with 3D objects through raycasts.

![](/static/images/tutorials/quick-setup-4.jpg)