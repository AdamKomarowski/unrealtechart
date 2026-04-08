---
title: "Static Meshes"
excerpt: ""
permalink: "/book/unrealengine/static-meshes/"
toc: true
toc_sticky: true
---

# Material Slots

Keep the number of material slots as low as possible. Each material slot generates a separate draw call, which increases rendering cost.

**Without Nanite:** minimize material slots as much as possible — merge materials where you can.

**With Nanite and translucent materials:** Nanite does not support translucency. If a mesh contains a translucent element, split it into two separate meshes:
- Opaque part — Nanite enabled.
- Translucent part — Nanite disabled.

Combine them into a single composed asset inside a **Blueprint**.

# Nanite Settings

## Enable Nanite Support

Enable Nanite Support should be **on** when your project uses Nanite, with the following exceptions:

- Mesh has **fewer than ~1000 triangles** — at this polygon count Nanite provides no benefit and adds overhead.
- Mesh contains **translucent materials** — Nanite does not support translucency; disable Nanite on the translucent part (see Material Slots above).

# LOD

Do **not** use LODs when Nanite is enabled — Nanite handles its own internal level-of-detail automatically.

Use LODs when:
- Nanite is **not** used in the project.
- Working with **Skeletal Meshes** — Nanite is not supported for skeletal meshes, so LODs should always be set up when Nanite is unavailable.

# Generate Lightmap UVs

**Disable** Generate Lightmap UVs unless you are baking static lighting (Lightmass).

From Unreal Engine 5 onward, the default lighting model is fully dynamic (Lumen). Generating lightmap UVs for meshes that will never be baked wastes memory and increases asset build times with no visual benefit.
