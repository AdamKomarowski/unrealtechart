---
title: "Textures"
excerpt: ""
permalink: "/book/unrealengine/textures/"
toc: true
toc_sticky: true
---

For a full reference on texture formats and settings, see the [official documentation](https://dev.epicgames.com/documentation/unreal-engine/texture-format-support-and-settings-in-unreal-engine).

## Texture Resolution

Always use **power-of-two** dimensions: 64, 128, 256, 512, 1024, 2048, 4096, etc. Textures do not have to be square — mixing powers of two is fine (e.g. 1024×512).

Power-of-two resolutions are required for:
- **GPU compression** — non-power-of-two textures cannot be compressed, which means they take significantly more GPU memory.
- **Mipmaps** — Unreal generates mip levels by halving the resolution at each step; this only works correctly with power-of-two sizes.
- **Streaming** — the texture streaming system relies on mip levels to load the appropriate resolution at runtime.

## Compression

Choosing the right compression setting reduces GPU memory usage without visible quality loss. In Unreal Engine the compression is set per texture in the texture editor.

| Compression Setting | Format | Use case |
|---|---|---|
| **Default** | BC1 / BC3 | Color / albedo textures (BC1 without alpha, BC3 with alpha) |
| **Normalmap** | BC5 | Normal maps — stores only R and G channels, B is reconstructed |
| **Masks** | BC4 | Single-channel masks — roughness, metallic, ambient occlusion |
| **HDR** | BC6H | HDR environment textures |
| **UserInterface2D** | — | UI textures, no mipmaps generated |

**Rule of thumb:** never leave a texture on Default compression if it is a normal map or a single-channel mask — you waste memory and may get rendering artifacts.

## Texture Group

The **Texture Group** (also called LOD Group) controls how Unreal streams and manages a texture at runtime. It sets the streaming priority, mipmap bias, and maximum allowed resolution for a group of textures.

Setting the correct Texture Group ensures:
- The streaming system loads the right resolution at the right time.
- High-priority surfaces (characters, weapons) keep full resolution longer than background props.
- You can cap resolutions per group in the device profile, making it easy to scale quality across platforms without touching individual assets.

Common Texture Groups:

| Group | Typical use |
|---|---|
| **World** | Environment / architectural surfaces |
| **WorldNormalMap** | Normal maps for environment meshes |
| **Character** | Character skin and clothing |
| **CharacterNormalMap** | Normal maps for characters |
| **Effects** | VFX textures, particles |
| **UI** | Interface elements |

Always assign the most specific group that matches the texture's purpose — do not leave everything on the default **World** group.
