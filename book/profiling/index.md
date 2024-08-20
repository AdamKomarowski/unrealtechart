---
title: "Tools"
excerpt: ""
permalink: "/book/tools/"
---

{% include toc icon="columns" title=page.title %}

{% include custom/wip-warning.md %}

{% include figure image_path="/assets/images/viewmode_quad_overdraw.jpg" alt="" caption="__Figure:__ Quad Overdraw view mode" %}

Since at least two generation of consoles, real-time graphics performance is no longer just a matter of pure polygon count. Features that are fundamental to physically based rendering - from specular reflections, through diffuse lighting to ambient occlusion - are largely decoupled from the cost of geometry, being dependent on the amount of pixels instead. Other issues - like quad overshading or texture bandwidth - need more attention now, as graphics chips became great at processing polygons, causing the bottlenecks to appear elsewhere.

To understand profilers' output, we need the knowledge about engine's inner workings. With popular commercial engines like Unreal Engine 4, understanding performance factors is at the same time hard and easy.

It's hard because engine's renderer consists of dozens of systems. These pieces of code are often dependent on each other, making it harder to isolate certain features for testing. By default, Unreal needs to go through 46 different [rendering passes](/book/profiling/lighting/) to process and draw a scene every frame. Usually, detailed explanation of these processes can only be found in engine programming documentation or by reading the source code. This makes the knowledge quite obscure to people without a background in engine programming, not just to artists.

But understanding performance issues is also easy now. The engine provides us with detailed statistics about each rendering pass and subsystem. Immediately useful information, including a frames-per-second graph, can be displayed with basic console commands. Various built-in tools (like the [GPU Visualizer](/book/profiling/gpu-visualizer/)) allow us to make a performance analysis - the step called _profiling_. And if we ever a need a precise breakdown of a single frame, we can turn to powerful [standalone applications](/book/profiling/external/). This book walks you through the full spectrum of tools that can be used to analyze performance.

