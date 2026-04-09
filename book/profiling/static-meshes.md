---
title: "Static Meshes"
excerpt: ""
permalink: "/book/unrealengine/static-meshes/"
toc: true
toc_sticky: true
---

## Material Slots

Keep the number of material slots as low as possible. Each material slot generates a separate draw call, which increases rendering cost.

**Without Nanite:** minimize material slots as much as possible — merge materials where you can.

**With Nanite and translucent materials:** Nanite does not support translucency. If a mesh contains a translucent element, split it into two separate meshes:
- Opaque part — Nanite enabled.
- Translucent part — Nanite disabled.

Combine them into a single composed asset inside a **Blueprint**.

## Nanite Settings

### Enable Nanite Support

Enable Nanite Support should be **on** when your project uses Nanite, with the following exceptions:

- Mesh has **fewer than ~1000 triangles** — at this polygon count Nanite provides no benefit and adds overhead.
- Mesh contains **translucent materials** — Nanite does not support translucency; disable Nanite on the translucent part (see Material Slots above).

## Fallback Target

Set Fallback Target to approximately **10% of the mesh's total polycount**.

The fallback mesh is a coarse representation of the high-detail Nanite mesh. It is used in situations where Nanite rendering is not supported or not ideal:

- When Nanite rendering is unavailable on the target platform.
- Complex collision generation.
- Baked lighting with lightmaps.
- Hardware ray tracing reflections with Lumen.

Because the fallback mesh is not the primary rendered representation, keeping it at ~10% polycount is a good balance between fidelity in fallback scenarios and memory/performance cost.

This matters especially for ray tracing with Lumen. Hardware Ray Tracing (HWRT) traces rays against the fallback mesh directly, so its quality depends on how well the fallback represents the original shape. Software Ray Tracing (SWRT) works differently — it traces against a Distance Field instead. SWRT also has two modes: Detail Tracing uses the Mesh Distance Field (MDF) which is per-object and more accurate, while Global Tracing uses the Global Distance Field (GDF) which is a lower-resolution scene-wide representation and therefore the least accurate option for indirect lighting.

**Tip:** You can preview how your Nanite fallback mesh looks in the viewport via **Show → Nanite → Nanite Fallback** (Fallback Mesh Visualization).

## LOD

Do **not** use LODs when Nanite is enabled — Nanite handles its own internal level-of-detail automatically.

Use LODs when:
- Nanite is **not** used in the project.
- Working with **Skeletal Meshes** — Nanite is not supported for skeletal meshes, so LODs should always be set up when Nanite is unavailable.

For guidance on how to tune LOD Screen Size per platform, see the [official documentation](https://dev.epicgames.com/documentation/unreal-engine/optimizing-lod-screen-size-per-platform-in-unreal-engine).

## Generate Lightmap UVs

**Disable** Generate Lightmap UVs unless you are baking static lighting (Lightmass).

From Unreal Engine 5 onward, the default lighting model is fully dynamic (Lumen). Generating lightmap UVs for meshes that will never be baked wastes memory and increases asset build times with no visual benefit.

## UV Channels

Keep the number of UV channels as low as possible — fewer UV channels means lower memory usage and faster rendering.

**Watch out when exporting from Maya:** Maya often generates additional UV channels automatically during export (e.g. for light maps or secondary UVs), even when they are not needed. Always check the UV channels in Unreal after import and remove any that are unnecessary.

## Collision

Keep collisions as simple as possible — the simpler the better. Prefer primitive shapes:

- **Sphere** — cheapest collision, ideal for rounded objects.
- **Box** — good for rectangular shapes.
- **Capsule** — useful for characters and vertical objects.

**Never use "Use Complex Collision as Simple"** — this uses the full render mesh for physics, which is extremely expensive and should be avoided in all cases.

If a mesh does not require any physical interaction (background props, purely visual elements), **do not assign any collision at all**.
