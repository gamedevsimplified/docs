---
order: 1000
title: Clone a template
---
!!!
This is a manual process at the moment but will be automated in a future update.
!!!
!!!
We'll be cloning `Basic` template in this example and naming it `MyGame`.
!!!

If you are basing your work on one of the templates (demos), I recommend you clone it to avoid eventual conflicts when upgrading to a newer version of the package. It is also handy to keep the original version as a reference.

---
### 1. Create a new folder

Create a new folder in `Assets` and name it `MyGame`.

---
### 2. Copy files

Go to `GDS/Demos/Basic` and copy everything except **Editor**, **Prefab**, and the **Scenes** into `MyGame`.

![](/static/images/tutorials/clone-template-2.jpg){.rounded-lg}

---
### 3. Update the Assembly Defnition Asset

Go to  `MyGame` and change the asset name and the assembly name in the inspector from `GDS/Demos/Basic` to `MyGame`. 

Don't forget to press **Apply**!

![](/static/images/tutorials/clone-template-3.jpg){.rounded-lg}

---
### 4. Update the namespace

<!-- In an editor of your choice, batch rename the old **namespace** to fix the naming conflicts (from `GDS.Demos.Basic` to `MyGame`). -->

To fix the naming conflicts, rename the existing **namespace** `GDS.Demos.Basic` to something appropriate for your project (`MyGame` in this example). Most IDEs offer batch renaming tools that safely update all references.

!!!
Make sure to limit the search scope to `MyGame` folder only.
!!!

![](/static/images/tutorials/clone-template-4.jpg){.rounded-lg}

---
### 5. Rename Assets and Folders
- Rename **Panel Settings** and the **Visual Tree** assets to `MyGame`.
- Go to `Resources` and rename `Basic` folder to `MyGame`
- Go to `Resources/MyGame` and rename `BasicTheme` to `MyGameTheme`

![](/static/images/tutorials/clone-template-5a.jpg){.rounded-lg}

![](/static/images/tutorials/clone-template-5b.jpg){.rounded-lg}

---
### 6. Update Panel Settings stylesheet

Go to `MyGame`, select the **Panel Settings** asset and update the **Theme Style Sheet** with `Resources/MyGame/MyGameTheme`

![](/static/images/tutorials/clone-template-6.jpg){.rounded-lg}

---
### 7. Create a new Scene

Create a new **Scene** and name it `MyGame`. 

---
### 8. Add an Inventory

Continue by following [this tutorial](/quick-start/manual-setup) to add an inventory to a scene.

