---
title: "Profiling"
excerpt: ""
permalink: "/book/unrealengine/profiling/"
toc: true
toc_sticky: true
---

A good starting point for learning profiling in Unreal Engine is this [profiling tutorial](https://www.youtube.com/watch?v=vnbARZHccpQ).

## ProfileGPU

`ProfileGPU` is a console command that captures a single-frame GPU snapshot and shows a breakdown of every rendering pass and its cost. It gives you an immediate overview of what is happening on the GPU — which passes are the most expensive and where the bottleneck is.

Run it from the console:

```
ProfileGPU
```

### Material Draw Events

By default ProfileGPU shows pass names but not individual material names. To see exactly which materials are being drawn, run these two commands first:

```
r.ShowMaterialDrawEvents 1
r.RHICmdBypass 0
```

After that, the next `ProfileGPU` capture will label each draw call with the material name, making it much easier to identify expensive or unexpected materials.

## MemReport

`MemReport` generates a full memory report — a detailed log of everything currently loaded into memory, including textures, meshes, sounds, and other assets. It works both on PC and on consoles, making it essential for tracking down memory budgets.

Run it from the console:

```
MemReport -full
```

The report is saved to the `Saved/Profiling/` folder. Use it to:

- See which assets are loaded and how much memory they consume.
- Identify textures streaming at full resolution when they shouldn't be.
- Catch assets that are loaded but no longer referenced.
- Verify memory usage before and after optimizations.

{% capture tutorialvideo %}A68wGA3CY_A?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}
