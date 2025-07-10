---
title: "Tools"
excerpt: ""
permalink: "book/tools/"
---

Building Tools in Unreal Engine 5

Welcome to the dedicated hub for tool creation in Unreal Engine 5. To make the learning path as clear—and actionable—as possible, this section is divided into three progressive parts, each building on the lessons of the previous one.

# Part 1 — Blueprints

We start with UE5’s powerful visual‑scripting system. You’ll learn how to prototype ideas rapidly, expose parameters for designers, and iterate without recompiling. Expect hands‑on demos that get results on‑screen fast.

Video Tutorials

# How to Use the BugIt Command in Unreal Engine

{% capture tutorialvideo %}Ntdtf0UY70U?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

# Tool: Generate Lightmap UVs

{% capture tutorialvideo %}S8XLTxFsikA?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

# Tool: Cast Shadows

{% capture tutorialvideo %}FwKKHfStsMg?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

# Tool: Empty Actors

{% capture tutorialvideo %}3-jSbxYF1W8?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

# Tool: Duplicated Actors

{% capture tutorialvideo %}rABhgArqbEU?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}


# Part 2 — Python

With the workflow proven in Blueprints, we’ll graduate to Python scripting. Here we automate repetitive tasks, build pipeline helpers, and interface with external tools. Python lets us glue systems together and speed up daily production chores.

To trigger a Python script directly from an Editor Utility Widget, simply use the Execute Python Script node.

{% include figure image_path="/assets/images/Tools/Python/PytonExecute.png" alt="Execute Python Script node in Unreal Engine" %}

# Tool: Organize Actors into Folders

{% capture tutorialvideo %}eg0ZIjq-25o?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

<div class="notice--info" markdown="1">
Code:

```python
import unreal

# Create instances for editor utilities
editor_level_lib  = unreal.EditorLevelLibrary()
editor_filter_lib = unreal.EditorFilterLibrary()

# Get all actors in the current level
all_actors = editor_level_lib.get_all_level_actors()

# Helper to filter actors by class
def filter_by_class(actors, cls):
    return editor_filter_lib.by_class(actors, cls)

# Filter actors by type
mapping = {
    "StaticMeshActors":     filter_by_class(all_actors, unreal.StaticMeshActor),
    "SkeletalMeshActors":   filter_by_class(all_actors, unreal.SkeletalMeshActor),
    "Brush":                filter_by_class(all_actors, unreal.Brush),
    "BlockingVolume":       filter_by_class(all_actors, unreal.BlockingVolume),
    "DecalActor":           filter_by_class(all_actors, unreal.DecalActor),
    "Landscape":            filter_by_class(all_actors, unreal.Landscape),
    "NavModifierVolume":    filter_by_class(all_actors, unreal.NavModifierVolume),
    "ReflectionCapture":    filter_by_class(all_actors, unreal.ReflectionCapture),
    "Blueprints":           editor_filter_lib.by_id_name(all_actors, "BP_"),
    "Lights":               filter_by_class(all_actors, unreal.Light),
}

moved_count = 0

# Move actors into their corresponding folders if not already in a folder
for folder_name, actors in mapping.items():
    unreal.log(f"{folder_name}: {len(actors)} actors")
    for actor in actors:
        if actor.get_folder_path() != '':
            continue  # skip if actor already has a folder
        actor.set_folder_path(folder_name)
        unreal.log(f"Actors moved {actor.get_fname()} to {folder_name}")
        moved_count += 1

# Move all actors without folders into 'Others'
others = [actor for actor in all_actors if actor.get_folder_path() == '']
if others:
    unreal.log(f"Others: {len(others)} actors")
    for actor in others:
        actor.set_folder_path("Others")
        unreal.log(f"Moved {actor.get_fname()} to Others")
        moved_count += 1

unreal.log(f"Moved {moved_count} actors into folders (including 'Others')")
```
</div>

# Tool: Spawn Skeletal Meshes by LOD

{% capture tutorialvideo %}eg0ZIjq-25o?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

<div class="notice--info" markdown="1">
Code:

```python
import unreal

# Helper: Get the number of LODs for a Skeletal Mesh
def get_lod_count(skel_mesh: unreal.SkeletalMesh) -> int:
    return len(skel_mesh.get_editor_property("lod_info"))

# Get the bounding box size (AABB) of a Skeletal Mesh
def get_mesh_size(skel_mesh: unreal.SkeletalMesh) -> unreal.Vector:
    bounds = skel_mesh.get_bounds()
    return bounds.box_extent * 2

# Place Skeletal Meshes with all LODs in a grid
def place_skeletal_meshes_along_axis(
    start_location=(0, 0, 0),
    selected_assets=None,
    x_spacing=100.0,
    y_spacing=500.0
):
    if selected_assets is None:
        selected_assets = unreal.EditorUtilityLibrary.get_selected_assets()

    location = unreal.Vector(*start_location)
    first_mesh = True

    for asset in selected_assets:
        if not isinstance(asset, unreal.SkeletalMesh):
            continue

        mesh_size = get_mesh_size(asset)
        lod_count = get_lod_count(asset)

        # Reset X position for each new asset (new row)
        location.x = start_location[0]

        for lod in range(lod_count):
            # Spawn the very first LOD of the first mesh at (0, 0, 0) for reference
            if first_mesh:
                location = unreal.Vector(0, 0, 0)
                first_mesh = False

            # ---- Spawn Skeletal Mesh Actor ----
            actor = unreal.EditorLevelLibrary.spawn_actor_from_class(
                unreal.SkeletalMeshActor, location
            )

            sk_comp = actor.get_component_by_class(
                unreal.SkeletalMeshComponent.static_class()
            )

            # Assign the mesh and force specific LOD
            if sk_comp:
                sk_comp.set_editor_property("skeletal_mesh", asset)

                if hasattr(sk_comp, "set_forced_lod_model"):  # UE 5.3+
                    sk_comp.set_forced_lod_model(lod + 1)     # 0=auto, 1=LOD0, etc.
                else:
                    sk_comp.set_editor_property("forced_lod_model", lod + 1)

            # Move right for the next LOD
            location.x += mesh_size.x / 2 + x_spacing

        # Move down after all LODs of the current mesh
        location.y += y_spacing

# inputs
place_skeletal_meshes_along_axis(
    start_location=(0, 0, 0),
    selected_assets=unreal.EditorUtilityLibrary.get_selected_assets(),
    x_spacing=200.0,
    y_spacing=1000.0
)
```
</div>

# Tool: Spawn Static Mesh Actors form Smallest to Biggest

{% capture tutorialvideo %}eg0ZIjq-25o?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

<div class="notice--info" markdown="1">
Code:

```python

import unreal

# -------------------------------------------------------------
# Helper: Returns the bounding box size of a static mesh
# -------------------------------------------------------------
def get_mesh_size(mesh):
    bounding_box = mesh.get_bounding_box()
    return bounding_box.max - bounding_box.min

# Helper: Returns the number of LODs for a static mesh
def get_lod_count(mesh):
    return mesh.get_num_lods()

# Main: Places selected meshes in order from smallest to largest,
#       spawns all their LODs in a row along the X-axis
def place_meshes_along_axis(start_location, selected_assets, spacing):
    previous_mesh_size = unreal.Vector(0, 0, 0)
    location = unreal.Vector(*start_location)
    is_first_mesh = True

    # Sort selected assets by volume (X * Y * Z size)
    sorted_assets = sorted(
        selected_assets,
        key=lambda asset: get_mesh_size(asset).x * get_mesh_size(asset).y * get_mesh_size(asset).z
    )

    for asset in sorted_assets:
        if not isinstance(asset, unreal.StaticMesh):
            continue

        mesh_size = get_mesh_size(asset)
        lod_count = get_lod_count(asset)

        for lod in range(lod_count):
            if is_first_mesh:
                # Place the first mesh at the starting position
                is_first_mesh = False
            else:
                # Move to the next position considering size and spacing
                location.x += previous_mesh_size.x / 2 + mesh_size.x / 2 + spacing

            # Spawn mesh actor in the level
            actor = unreal.EditorLevelLibrary.spawn_actor_from_object(asset, location)

            # Force this actor to use a specific LOD
            sm_component = actor.get_component_by_class(unreal.StaticMeshComponent.static_class())
            if sm_component:
                sm_component.set_forced_lod_model(lod)

            # Offset for the next LOD instance
            location.x += mesh_size.x / 2 + spacing
            previous_mesh_size = mesh_size

# Inputs
input_x = 0      
input_y = 0       
input_z = 0        
spacing = 50      

# Get selected assets from the Content Browser
selected_assets = unreal.EditorUtilityLibrary.get_selected_assets()

# Run the main function with provided inputs
place_meshes_along_axis([input_x, input_y, input_z], selected_assets, spacing)


</div>


# Part 3 — C++

Finally, we translate our prototypes into performant, production‑ready C++. This chapter focuses on writing clean modules, extending the editor, and squeezing every drop of performance out of custom tools.

Whether you’re a technical artist looking to iterate visually, a pipeline engineer automating processes, or a programmer chasing raw speed, you’ll find something valuable in each chapter. Dive in, follow along, and let’s build better tools together!