---
title: "CPU Profiling"
excerpt: ""
permalink: "/book/process/measuring-performance/"
---

In this chapter you'll learn about:

* How to use Unreal Insights
* Profile CPU

## CPU or GPU Bound?

__Identifying bottlenecks:__

 Use `stat unit`, not just `stat fps`.
  Largest number shows you the likely bottleneck.
  You can also use stat unitGraph, which shows a line graph playback. Mostly useful for spotting repeating hitches.

__Frame:__ total time to finish each frame.

__Game:__ C++ or BP gameplay operations.

__Draw:__ CPU render time.

__GPU:__ GPU Render time.

If possible, avoid profiling your game in the editor.
Prefer cooked builds, running on the target platform. But if you have to do it, always:

- Play in Standalone
- Minimize the Editor
- Make sure to turn off frame Rate Smoothing(Project Settings)
- Turn off VSync (r.vsync 0)


__Analysis of a frame:__

Before Rendering:

CPU: Game (Game context) -> Draw (What to render)

GPU: GPU (Final pixels)

Generate a Chart Over a Period of Time. 

Very useful to get the stat unit times over a longer period of time.

`StartFPSChart` and `StopFPSChart`

Results in a .csv file that can be used to be potted in a chart. 
(eg. in game cutscene or a camera path set up for automated tests).

## Profiling CPU

Profiling can be used to capture information causing __bottlenecks__ or slowdowns in both the CPU and GPU.

`stat Startfile` // To start

`stat Stopfile` // To stop

These captures can be loaded and analyzed through the __Unreal Frontend__ in the Editor. 

Window -> Developer Tools -> Session Frontend

* __In addition it is worth to add:__

`-toggledrawevents -statnamedevents -trace=cpu,frame,gpu,LoadTime,file,memory,net,Bookmark,log`

## Unreal Insights 

__Standalone__ profiling system. Makes it easy to add __your own profiling data.__ 
Finally, it can __record data remotely.__ __Smaller impact__ on execution.

__Game__ thread: Calculates all __game logic__ and __transforms.__ 
* Animations
* Object Positions
* Caracter Movement
* Physics
* Spawning
* AI

By default, Ticks are calculated __every frame.__ The fact is, Blueprints __rarely__ need to tick by default. (Ticking can be disabled per-Blueprint in the Blueprint Class Defaults)
Too many actors ticing can __substantially slow down__ a project and identifying these is important. 

`stat game` gives a general idea on how long the various gameplay ticks are taking. 
Use `dumpticks` to see a list of all actors that are currently ticking. 

Which of these could be driven by __Events__ or __Timers__?

__Alternatives__ to Tick:
* Timelines
* Timers
* Manual toggling of Actor Tick
* Reducing the Tick Interval
* Event driven systems (use dispatchers!)

Simple rotation effects can be replaced with native logic or shaders. Using __RotatingMovementComponent__ can do work in C++. 

Or use __RotateAboutWorldAxis__ material function to calculate in on the vertex shader.

* __Expensive__ Functionality: Try to avoid using inherently __expensive__ functions: 

- Get All Actors of Class (Functions like that can wreak havoc on performance)

- For Loop (Can be expensive especially when nested. Consider using breakable loops if iterations aren't necessary after a result has been found.)

- SpawnActor NONE (The more complicated construction script is, the more extensive the spawn time.)

Convert __Complex__ functionality to C++. It is good practice to move complex logic to native and expose them to Blueprints. 

Animation __Fast__ Path: Make sure Anim Graph uses Fast path for property access.


__Draw__ thread

__Frustrum__ culling checks what is in front of the camera, while __hardware occlusion__ queries based on the scene depth buffer.
Too many objects(10-15k+) can cause __performance__ impact.

Critical on games with __open/large__ environements.

__Draw__ calls

Draw calls have a __huge__ impact on performance.

* You can check amount of drawcall by: stat scenrendering
__Conclusion:__ We can say as the number of draw inches increases, the frame rendering time increases.

Each time the renderer is done it needs to receive commands from the render thread, which adds __overhead__.
Draw calls have a much __bigger impact than polycount__ in many scenarios.

__Reducing__ draw calls

To lower the draw calls, it is better to use __fewer larger models__ than many small ones.
However, you cannot do that too much, as it __impactis everything else__ negatively:

- Worse for occlusion

- Worse for lightmapping

- Worse for collision calculation

- Worse for memory

Level of Details (LODs):

- __Simplifies__ a model or buch of models in given conditions.

- Essentialy __swaps__ one model for another simpler model.

- Usually means a model becomes __lower poly__ in the distance.

- Can also be used to __switch__ between multi-material meshes to a single material.

Using __modularity__ to build levels is a common technique. It saves work time and memory. 
However, this __increases__ draw calls. If using a modular mesh workflow, you can always __merge meshes__ later on if needed.

You can also do __instanced__ Rendering.

- Automatically __groups__ models together into single draw calls.

- 4.21+ does a great job at __auto-instancing__ similar meshes, but it's not a definitive solution.

- In some cases, __manually__ setting up instanced meshes is ideal.

__Hierarchical__ Level of Detail (HLODs):

- __Gropus__ models together to reduce draw calls.

- __Combine__ materials and textures (atlas)

- Automatically replace __multiple__ models with combined Static meshes.

- 100% __non-destructive__.




## GPU Profiling



__WIP__