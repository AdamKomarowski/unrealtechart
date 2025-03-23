---
title: "Unreal Engine 5"
excerpt: ""
permalink: "/book/unrealengine/"
---

## Lumen in Unreal Engine 5.5 

Lumen is Unreal Engine 5's fully dynamic global illumination and reflections system that is designed for next-generation consoles, and it is the default global illumination and reflections system. Lumen renders diffuse interreflection with infinite bounces and indirect specular reflections in large, detailed environments at scales ranging from millimeters to kilometers.

Lumen (luminous flux): 
Lumen is a total amount of light emitted by a source in all directions.

Lux (illuminance flux):
Lux is equal to one lumen per sqare meter and is the total amount of light that falls onto a surface. 

## LIGHTS:

In video games, light is simulated as rays that travel from a light source. Game engines like Unreal Engine use techniques such as ray tracing to accurately render reflections, refractions, and shadows. While light in reality is an electromagnetic wave, in computer graphics, it is treated as straight lines—rays—that determine how light interacts with objects in the scene.

In the context of Unreal Engine 5, Lumen is an advanced Global Illumination (GI) and ray-traced reflections system that dynamically simulates light behavior in real-time. Regarding ray tracing, Lumen operates in two main modes:

## Lumen has two Ray Tracing modes:

📊 1. Software Ray Tracing (Software RT)
Description: This mode does not require dedicated ray tracing hardware (it works on GPUs without RT cores). It uses screen-space techniques and voxelization to simulate reflections and light propagation.

Advantages:

Works on a wider range of hardware (including consoles like PlayStation 5 and Xbox Series X).

Better performance compared to hardware ray tracing.

Disadvantages:

Less accurate reflections (limited to visible objects—prone to screen-space artifacts).

Reduced precision in complex scenes with multiple reflections.

💻 2. Hardware Ray Tracing (Hardware RT)
Description: This mode fully utilizes ray tracing hardware (e.g., NVIDIA RTX or AMD RDNA 2) to perform ray calculations.

Advantages:

More accurate reflections (including objects off-screen).

Realistic indirect lighting (e.g., color bleeding from reflective surfaces).

Disadvantages:

Higher GPU workload.

Requires modern GPUs with ray tracing support.

🎯 How Does Lumen Relate to Ray Tracing?
Adaptive Approach: Lumen automatically chooses the appropriate technique based on settings and hardware—using Software RT when possible and switching to Hardware RT for more demanding effects.

Hybrid Modes: You can mix modes, e.g., use Software RT for Global Illumination and Hardware RT for Reflections to balance performance and quality.

🔧 Practical Tip:

If you prioritize performance, stick with Software Ray Tracing.

For the highest quality, enable Hardware Ray Tracing and fine-tune the Quality Level in the Lumen settings.







HOW WE SHOULD THINK TO WORK WITH LUMEN General Pipeline

Wat we should avoid with Lumen workflow

USEFULL COMMANDS

COMMON PROBLEMS AND FIX



References:

https://www.youtube.com/watch?v=nlbJwMoj1Dg&ab_channel=UnrealEngine

https://dev.epicgames.com/documentation/en-us/unreal-engine/lumen-global-illumination-and-reflections-in-unreal-engine
