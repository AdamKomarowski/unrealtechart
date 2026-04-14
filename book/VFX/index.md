---
title: "Tools"
excerpt: ""
permalink: "book/vfx/"
toc: true
toc_sticky: true
---

## Optimize VFX

## Emitters — Less Is Faster

Every emitter is a separate object updated each frame. More emitters = more VM invokes or GPU dispatches per frame. Merging emitters reduces the cost of spinning up and shutting down simulations — it does **not** directly reduce component counts or `TickComponent` calls (important clarification from the Epic team).

**When to merge:**
- Sprite + Mesh with the same spawn logic → OK
- Many similar effects (sparks, dust) → one emitter with different parameters

**When to keep separate:**
- Ribbon / Beam (requires separate sorting)
- Mixing CPU and GPU emitters
- Very different lifecycle logic

**Approximate costs:**

| Emitter Count | Niagara CPU (ms) | Draw Calls |
|---|---|---|
| 1–5 | ~0.3–0.6 | 1–2 |
| 10–15 | ~0.8–1.2 | 3–5 |
| 30 | ~1.5–2.5 | 6–10 |

**GPU vs CPU emitters:** GPU pays off only at 1000+ particles. Below that, CPU emitters are faster — GPU simulation has its own startup overhead.

---

## Lightweight Emitters (UE 5.6+)

An experimental feature that reduces CPU cost by stripping complex features. As of UE 5.6, Lightweight Emitters support Ribbon, Mesh Renderer, Sprite, Decal, and Light.

| | Standard | Lightweight |
|---|---|---|
| CPU (ms, 10 emitters) | ~1.2 | ~0.4 |
| GPU (ms) | ~0.6 | ~0.3 |
| Custom modules | ✅ | ❌ |
| User Parameters | ✅ | ❌ |

Good for simple mass effects (sparks, background fog). Avoid when you need per-particle logic or advanced events.

---

## Materials and Draw Calls

> **Important:** Auto-batching only works for **static draws**. Niagara uses dynamic draws — batching won't happen automatically even with the same material. To reduce draw calls, push more particles into a single system using data channels.

### What breaks batching (and what doesn't):

| | Breaks draw call? |
|---|---|
| Scalar / Vector Parameter | ❌ No |
| Texture Atlas (same asset) | ❌ No |
| Static Switch / Static Bool | ✅ Yes |
| Separate texture asset | ✅ Yes |
| Different Blend Mode | ✅ Yes |

**Practical rules:**
- One Master Material per category: `MFX_Sprite_Master`, `MFX_Ribbon_Master`, `MFX_Distortion`
- Instead of Static Switch use `Lerp` or `If` — compiles one shader instead of two variants
- Drive color/UV/brightness changes through Dynamic Material Instances (Scalar/Vector parameters)
- Texture Atlas still makes sense on mobile; on PC/console individual textures at 512–1024px with a shared master material is the standard approach

---

## Shader Complexity and Overdraw

### Pixel Shader instruction limits — rough guidelines:

| Platform | Recommended | Max |
|---|---|---|
| PC High-End | 250–300 | 400+ |
| PS5 / XSX | 250–350 | 400+ |
| PS4 / Xbox One | 180–220 | 250–280 |
| Mobile | 80–120 | 150–180 |
| VR/AR | 100–150 | 200 |

Instruction count alone doesn't decide performance — **screen coverage** matters just as much. A 300-instruction material on a tiny spark is fine; the same material on a full-screen smoke is a problem.

### Expensive nodes:

| Node | Approximate Cost |
|---|---|
| SceneColor | +40–80 instr. |
| Refraction | +20–40 instr. |
| DepthFade / Distance | +15–30 instr. |
| Texture Sample | ~4–6 instr. |
| Multiply, Lerp | ~1–2 instr. |

### Overdraw

Overdraw = the same pixel rendered multiple times by overlapping translucent particles. Smoke or magic clouds can reach 5–20x overdraw easily.

**How to reduce it:**
- **Cutout texture** — instead of rendering a full rectangular sprite (mostly transparent), mask the shape. Fewer pixels to process
- `DepthFade` — hides particles too close to geometry, eliminating invisible overdraw
- Large effects = simple materials (max ~100–150 instructions)
- `SceneColor` / Refraction only when the effect covers a small screen area

### How to check:
- Viewport → **Shader Complexity** or **Quad Overdraw** (green = OK, red = warning, white = problem)
- `stat scenerendering` → PS instructions
- Material Editor → Stats → Instructions

---

## Profiling

Optimizing without profiling is shooting in the dark.

### Stat Commands

**`stat GPU`** — shows the total GPU impact per stage. Useful for spotting overdraw cost in translucent particles and cycle cost of Niagara GPU Simulation passes.

**`stat Niagara`** — a basic cost and memory overview: tick time, active system and emitter counts, total particle count, and memory usage. Good first stop when something feels slow.

**`stat NiagaraEmitters`** — per-frame timing broken down by individual emitters. Use this to identify which specific emitter is the bottleneck.

**`stat NiagaraSystems`** — per-frame timing broken down by Niagara Systems. Useful for comparing the cost of different systems against each other.

| Tool | What it shows |
|---|---|
| `stat GPU` | Total GPU impact — overdraw in translucency, Niagara GPU Simulation cost |
| `stat Niagara` | Basic cost and memory overview: tick time, counts, memory |
| `stat NiagaraEmitters` | Per-frame timing per emitter |
| `stat NiagaraSystems` | Per-frame timing per system |
| GPU Profiler (`Ctrl+Shift+,`) | `Niagara::Tick`, `Niagara::Render`, material costs |

### Niagara Tick Time budget:

| Platform | Acceptable Tick Time |
|---|---|
| PC High-End | < 2–3 ms |
| PS5 / XSX | < 2 ms |
| PS4 / Xbox One | < 1.5 ms |
| Mobile / VR | < 1 ms |

---

## Niagara Effect Type — Scalability Control Center

`Niagara Effect Type` is a global settings asset for culling, scalability, and performance budgets. Create it in Content Browser: **Add New → FX → Advanced → Niagara Effect Type**, then assign it to each Niagara System in the `Effect Type` field.

{% capture tutorialvideo %}VefnDwakcDs?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

### Core scalability settings:

| Setting | Description |
|---|---|
| Spawn Count Scale | Reduces particle count |
| Update Frequency | Fewer updates = less CPU/GPU load |
| Cull Distance | Disables effect beyond a distance |
| Max Instances | Limits active instances of this system |

### Budget settings:

| Setting | Description |
|---|---|
| Max Distance | Beyond this → effect disappears |
| Max Effect Type Instances | How many systems of this type can run simultaneously |
| Cull Proxy Mode | Replace with null or a simple proxy effect |
| Allow Pre Culling by View Frustum | Auto-disable when outside camera frustum |
| Max Time Without Render | How long an invisible effect can keep running |

### Budget Scaling (auto-degradation when overloaded):

| Setting | Effect |
|---|---|
| Max Global Budget Usage | Max GPU/CPU usage allowed (0.0–1.0) |
| Max Distance Scale by Budget | Reduces draw distance when overloaded |
| Max Instance Count Scale by Budget | Reduces particle count when overloaded |
| Max System Instance Count Scale by Budget | Reduces system count when overloaded |

### Significance Handler — what survives when over budget:

| Type | Priority Rule |
|---|---|
| Distance | Closer to camera = more important |
| Age | Newer effects = more important |
| Custom | Blueprint / C++ |

---

## Scalability Overrides per System

In each Niagara System → **Scalability** tab → Enable Overrides. Lets you control behavior per quality level (Epic / High / Medium / Low):

**Example — system with 3 emitters (Core, Smoke Trail, Sparkles):**

| Quality | Active emitters |
|---|---|
| Epic | Core + Smoke + Sparkles |
| High | Core + Smoke + Sparkles |
| Medium | Core + Smoke (Spawn Scale 0.5) |
| Low | Core only |

Instead of building separate systems per platform — put LOD logic directly into a single Niagara System. Global settings are read from `Niagara Effect Type`, but each system can override them locally via Scalability Overrides.

Useful console variables for testing: `r.NiagaraScalability.*`

---

## Lifecycle — Effects That Never End

One of the most common mistakes: an effect loops forever and eats resources even when nothing is visible.

**Best practices:**
- `Loop Behavior = Once` for one-shot effects (explosions, impacts)
- `Auto Deactivate = true`
- `Kill on Complete` per emitter
- Avoid empty emitters with an active loop

To verify: Niagara Debug HUD → System Overview → look for emitters running with zero particles.

---

## Fixed Bounds

By default Niagara recalculates the bounding box dynamically every frame. For static or predictable effects, set **Fixed Bounds** manually — eliminates per-frame calculation and stabilizes culling.
