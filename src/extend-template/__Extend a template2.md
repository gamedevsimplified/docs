---
order: 1000
---

!!!
Cloning a template is a manual process at this moment. It will be automated in a future update.

We'll be cloning `Basic` in this example and renaming it to `AwesomeGame`
!!!

If you are basing your work on one of the templates (demos), I recommend you clone it to avoid eventual conflicts when upgrading to a newer version. It is also handy to keep the original version as a reference.

1. Copy and paste `Basic` folder. Rename the copy to `AwesomeGame`.
---
2. Update the `Basic` Assembly Defnition Asset inside `AwesomeGame`:
- Change the file name and the assembly name in the inspector to `AwesomeGame`. 
- Don't forget to press **Apply**!

![](/static/images/tutorials/rename-assembly.jpg)

---
3. Update the `Basic.Editor` Assembly Definition Asset in `AwesomeGame/Editor`:
- Change the file name and the assembly name in the inspector to `AwesomeGame.Editor`. 
- Update the reference to the newly created `AwesomeGame` assembly (from `Basic`).
- Don't forget to press **Apply**!

![](/static/images/tutorials/rename-editor-assembly.jpg)

---
4. Batch rename the old **namespace** to fix the naming conflicts (from `GDS.Basic` to `AwesomeGame`).

![](/static/images/tutorials/batch-rename.jpg)

---
5. Rename the **Panel Settings**, **Visual Tree** and the **Scene** assets to `AwesomeGame` to avoid potential confusion.

---
6. Open the `AwesomeGame` scene and select the **Inventory** object in the scene tree (it should have a **UI Document** behavior attached ).

---
7. Update  **Panel Settings** and **Visual Tree** references in the **UI Document** behavior.
!!!
Since this is a clone, the **UI Document** fields are referencing the original assets.
Make sure to update the references to the new ones.
!!!

![](/static/images/tutorials/cloned-update-ref.jpg)

---
8. Play the Scene.
