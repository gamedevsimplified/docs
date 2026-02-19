---
order: 600
title: Troubleshooting 🚧
---

{{ include "snippets/wip" }}

### Discord Server

Join the [**Discord server**](https://discord.com/invite/MFgMX6Fn4m) for additional support from the developer and the community.

### Document doesn't look right in UI Builder

If you open a document in UI Builder and it looks like it doesn't have the right styling, make sure to select the right **theme** from the **dropdown**.

Each demo comes with its own theme, grid examples use **GDS Grid**, and the rest - **GDS Default**.

![](/static/images/troubleshoot-ui-builder.jpg)

In rare cases, themes may become corrupted during import. This is usually caused by an invalid asset path or GUID mismatch. If this occurs, please contact the developer for assistance.

### Objects in the scene do not react to pointer

- Add `Physics Raycaster` component to main camera.
- Add `EventSystem` component to scene hierarchy.
- Make sure the UI Document is not blocking input. Try setting the blocking element's **PickingMode** to **Ignore**.

![](/static/images/troubleshoot-picking-mode.jpg)

### Bag doesn't show up in the inspector

Make sure the class is marked as **Serializable**.