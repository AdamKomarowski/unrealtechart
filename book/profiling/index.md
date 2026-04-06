---
title: "Unreal Engine 5"
excerpt: ""
permalink: "/book/unrealengine/"
---

## Lumen in Unreal Engine 5.5

Lumen is Unreal Engine 5's fully dynamic global illumination and reflections system that is designed for next-generation consoles, and it is the default global illumination and reflections system. Lumen renders diffuse interreflection with infinite bounces and indirect specular reflections in large, detailed environments at scales ranging from millimeters to kilometers.

Lumen (luminous flux):
Lumen is the SI unit of luminous flux — the total amount of light emitted by a source in all directions.

Lux (illuminance):
Lux is equal to one lumen per square meter and represents the total amount of light that falls onto a surface.

## Lights

In video games, light is simulated as rays that travel from a light source. Game engines like Unreal Engine use techniques such as ray tracing to accurately render reflections, refractions, and shadows. While light in reality is an electromagnetic wave, in computer graphics it is treated as straight lines — rays — that determine how light interacts with objects in the scene.

In the context of Unreal Engine 5, Lumen is an advanced Global Illumination (GI) and reflections system that dynamically simulates light behavior in real-time. Lumen operates in two main ray tracing modes:

## Lumen has two Ray Tracing modes:

1. Software Ray Tracing (Software RT)

Description: This mode does not require dedicated ray tracing hardware (it works on GPUs without RT cores). It uses Mesh Distance Fields and screen-space techniques to simulate reflections and light propagation.

Advantages:
- Works on a wider range of hardware (including consoles like PlayStation 5 and Xbox Series X).
- Better performance compared to hardware ray tracing.

Disadvantages:
- Less accurate reflections (limited to visible objects — prone to screen-space artifacts).
- Reduced precision in complex scenes with multiple reflections.

2. Hardware Ray Tracing (Hardware RT)

Description: This mode fully utilizes ray tracing hardware (e.g., NVIDIA RTX or AMD RDNA 2) to perform ray calculations.

Advantages:
- More accurate reflections (including objects off-screen).
- Realistic indirect lighting (e.g., color bleeding from reflective surfaces).

Disadvantages:
- Higher GPU workload.
- Requires modern GPUs with ray tracing support.

**How Does Lumen Relate to Ray Tracing?**

Adaptive Approach: Lumen automatically chooses the appropriate technique based on settings and hardware — using Software RT when possible and switching to Hardware RT for more demanding effects.

Hybrid Modes: You can mix modes, e.g., use Software RT for Global Illumination and Hardware RT for Reflections to balance performance and quality.

Practical Tip:
- If you prioritize performance, stick with Software Ray Tracing.
- For the highest quality, enable Hardware Ray Tracing and fine-tune the Quality Level in the Lumen settings.

**Commands:**

Lumen RT method (SRT / HRT)
`r.Lumen.HardwareRayTracing 0–1`
- `0` : Software Ray Tracing
- `1` : Use Hardware Ray Tracing for Lumen when available, or Software Ray Tracing otherwise. HRT is more expensive.

Reflection Method
`r.ReflectionMethod 0–3`
- `0` : None
- `1` : Lumen
- `2` : SSR
- `3` : RT Reflections

Virtual Shadow Map On / Off
`r.Shadow.Virtual.Enable 0–1`
- `0` : Use old Shadow Map (Cascade shadows)
- `1` : Virtual Shadow Map

Ray Tracing features On / Off
`r.RayTracing.ForceAllRayTracingEffects -1–1`
- `-1` : Do not force (default)
- `0` : All ray tracing effects disabled
- `1` : All ray tracing effects enabled

- `r.RayTracing.Reflections 0–1`
- `r.RayTracing.Shadows 0–1`
- `r.RayTracing.AmbientOcclusion 0–1`

## Unreal Engine Lumen

{% include figure image_path="/assets/images/Lumen/LumenEnabled.png" alt="" caption="__Comments:__ Lumen Enabled" %}

{% include figure image_path="/assets/images/Lumen/ReflectionMethod.png" alt="" caption="__Comments:__ Reflection Method" %}

{% include figure image_path="/assets/images/Lumen/GenerateMeshDistanceField.png" alt="" caption="__Comments:__ Generate Mesh Distance Field" %}

{% include figure image_path="/assets/images/Lumen/RestartEditor.png" alt="" caption="__Comments:__ Restart Editor" %}

## How does it work?

{% include figure image_path="/assets/images/Lumen/CarsPreview.png" alt="" caption="__Comments:__ r.Lumen.Visualize.CardPlacement 1" %}

By default, Lumen only places 12 Cards on a mesh, but you can increase that amount by setting **Max Lumen Mesh Cards** in the Build Settings of the Static Mesh Editor. Adjusting the number of cards is useful for more complex interiors or single meshes with irregular shapes.

These projection planes or capture positions are called cards, and are generated offline for each mesh. Cards can be visualized with the console command `r.Lumen.Visualize.CardPlacement`. Using the card's projection, Lumen then renders the triangle meshes to capture all of the material properties from multiple angles. After the surface cache is populated with material properties, Lumen has all the volume and surface information it needs. Lumen then calculates direct and indirect lighting, and the result is fed back into the surface cache to be used in subsequent frames. These lighting updates are amortized over multiple frames, which is what allows Lumen to support many dynamic lights and multi-bounce global illumination.

{% include figure image_path="/assets/images/Lumen/LumenMesh.png" alt="" %}

After the Surface Cache is populated with material properties, Lumen has all the volume and surface information it needs. It then calculates direct and indirect lighting, and the results are fed back into the Surface Cache to be used in subsequent frames. These lighting updates are amortized over multiple frames, allowing Lumen to support many dynamic lights and multi-bounce global illumination.

It is important to note that volume and surface information are stored separately on the GPU. However, for simplicity, we refer to them as a single entity called the Lumen Scene.

You can visualize it using the Lumen Scene View mode. To do this, go to the Viewport and select:
Show → Visualize → Lumen Scene.

The Mesh Distance Field resolution is determined based on the imported scale of the static mesh.

If you apply significant scaling to placed instances, use the **Distance Field Resolution Scale** to compensate.

{% include figure image_path="/assets/images/Lumen/GenerateMeshDistanceField.png" alt="" caption="__Comments:__ Mesh Distance Fields Resolution Scale" %}

By default, Lumen traces against each mesh distance field for the first few meters for accuracy, and then traces the merged global distance field for the rest of each ray.

{% include figure image_path="/assets/images/Lumen/GlobalDistanceFields.png" alt="" caption="__Comments:__ Global Distance Fields" %}

This approach is used for objects farther from the camera, allowing traces to be much faster and less dependent on scene complexity.

Scenes that contain areas with many small objects packed together should consider baking these into a single mesh or setting the software tracing mode to **Global Traces**. This may reduce overall quality but will improve performance.

{% include figure image_path="/assets/images/Lumen/GDSF.png" alt="" caption="__Comments:__ Global Distance Fields" %}

{% include figure image_path="/assets/images/Lumen/GlobalTracing.png" alt="" caption="__Comments:__ Global Tracing" %}

- Lumen is disabled on Medium and Low scalability settings, which means a fallback setup should be provided for those platforms.

Dynamic GI Method
`r.DynamicGlobalIlluminationMethod 0–4`
- `0` : None. Global Illumination can be baked into Lightmaps but no technique will be used for Dynamic Global Illumination.
- `1` : Lumen. Use Lumen Global Illumination for all lights, emissive materials casting light and SkyLight Occlusion. Requires 'Generate Mesh Distance Fields' enabled for Software Ray Tracing and 'Support Hardware Ray Tracing' enabled for Hardware Ray Tracing.
- `2` : SSGI. Standalone Screen Space Global Illumination. Low cost, but limited by screen space information.
- `3` : RTGI. Ray Traced Global Illumination technique. Deprecated — use Lumen Global Illumination instead.
- `4` : Plugin. Use a plugin for Global Illumination.

- Lumen is not supported on previous-generation consoles, including PlayStation 4 and Xbox One.
- Mesh Distance Fields must be enabled for Lumen to work (`r.GenerateMeshDistanceFields=1`).

## Useful Commands

- `r.LumenScene.Radiosity.UpdateFactor` — controls how many indirect lighting passes are updated per frame. Higher values update more of the surface cache each frame at a higher GPU cost.
- `r.Lumen.Visualize.CardPlacement` — preview of the Mesh Cards
- `r.Lumen.Reflections.SmoothBias` — range from 0 to 1, where 0 means no change in roughness, and 1 makes it fully smooth.
- `r.Lumen.Reflections.ScreenSpaceReconstruction` — BRDF reweighting, values of 0 or 1 to disable/enable.
- `r.Lumen.Reflections.BilateralFilter` — denoising, values of 0 or 1 to disable/enable.
- `r.Lumen.Reflections.DownsampleFactor` — reduces the resolution of reflection rendering, values of 1/2/3/4, where 1 is full resolution, 2 is half, and so on.
- `r.Lumen.Reflections.MaxRoughnessToTraceClamp` — controls the maximum roughness value for reflections to be generated, default is 0.4. Lower values help reduce cost at the expense of fewer reflections. 0.3 is a good starting point.
- `r.Lumen.Reflections.MaxRoughnessToTraceForFoliage` — the same as above but for foliage with Two-Sided and Subsurface materials. Default is 0.4, lowering it to 0.2 is a decent trade-off.

## Common Problems and Fixes

- **Lumen flickering / noise:**
  1. Caused by emissive objects that are too small or too bright.
  2. Try increasing the attenuation radius of the light and reducing emissive intensity.
  3. Add eye adaptation to the emissive.

{% include figure image_path="/assets/images/Lumen/EyeAdaptation.png" alt="" caption="__Comments:__ Eye Adaptation" %}

- **Black meshes in Lumen Surface Cache:**

{% include figure image_path="/assets/images/Lumen/LumenBlackMeshes.png" alt="" caption="__Comments:__ Black Mesh" %}

These areas will not bounce light and will appear black in reflections. Issues like this can be fixed by increasing the number of Cards used with **Max Lumen Mesh Cards**, but that may not solve all cases. Alternatively, breaking the mesh into less complex pieces can resolve these types of issues as well.

## General requirements to work with Lumen

- Meshes divided into modules
- Correct Distance Fields
- Proper materials and texture brightness
- Optimal number of Mesh Cards
- Correct light setup
- Avoiding small and bright emissive surfaces
- Avoiding non-uniform scaling and scaling greater than 2

## Optimization

- **Custom Scalability**
- **Lumen Reflection Roughness Threshold Optimization:**

Involves controlling when accurate reflection rays are traced and when the system falls back to a simplified reflection model.

Key Points:
- `r.Lumen.Reflections.MaxRoughnessToTrace` — sets the roughness threshold above which pixels will use a rough specular approximation instead of tracing dedicated reflection rays. A lower value means more precise reflections but lower performance. A higher value means more approximations, which boosts performance but degrades reflection quality.
- `r.Lumen.Reflections.MaxRoughnessToTraceForFoliage` — a separate roughness threshold for materials using the Two Sided Foliage or Subsurface shading models (e.g., leaves, grass). Reflections on foliage are generally hard to see, so they can be optimized without significant quality loss. Setting this to `0` effectively disables ray tracing for reflections on foliage, which can significantly improve performance.

In summary: you can enhance Lumen's performance by limiting precise reflections on high-roughness materials, especially for foliage where reflections are less noticeable.

- **Replacing Lumen Reflections with Screen Space Reflections:**

Reflection costs can be more aggressively reduced by replacing Lumen Reflections with Screen Space Reflections (SSR). This can be achieved by setting `r.Lumen.Reflections.Allow=0`.

- **Surface Cache Tile Updates:**

Lumen Scene Lighting updates both the direct and indirect lighting in the surface cache. Performance depends on the portion of the surface cache updated each frame. You can adjust the update speed for direct and indirect lighting separately using `r.LumenScene.DirectLighting.MaxLightsPerTile` and `r.LumenScene.Radiosity.UpdateFactor`.

Lumen Scene Lighting selects a small subset of the most important lights for each surface cache tile, making its performance less sensitive to the total number of lights in the scene. The number of lights per tile can be controlled by `r.LumenScene.DirectLighting.MaxLightsPerTile`.

- **Profiling Lumen:**

{% include figure image_path="/assets/images/Lumen/LumenGPU.png" alt="" caption="__Comments:__ Lumen Profiling" %}

Use `stat GPU` in the viewport to check how much time Lumen takes per frame. The relevant categories are `LumenSceneLighting` and `LumenReflections`. For a more detailed breakdown, open the GPU Profiler with `Ctrl+Shift+,`.

---

References:

[Lumen](https://www.youtube.com/watch?v=nlbJwMoj1Dg&ab_channel=UnrealEngine)

[Lumen Global Illumination and Reflections in UE](https://dev.epicgames.com/documentation/en-us/unreal-engine/lumen-global-illumination-and-reflections-in-unreal-engine)

[Lumen in UE5](https://www.unrealengine.com/en-US/blog/lumen-in-ue5-let-there-be-light)

[Lumen Technical Details](https://dev.epicgames.com/documentation/en-us/unreal-engine/lumen-technical-details-in-unreal-engine?application_version=5.2)

[Lumen Performance Guide](https://dev.epicgames.com/documentation/en-us/unreal-engine/lumen-performance-guide-for-unreal-engine?application_version=5.2)

[Lumen Commands](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-console-variables-reference)
