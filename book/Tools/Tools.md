---
title: "Tools"
excerpt: ""
permalink: "book/tools/"
---

Building Tools in Unreal Engine 5

Welcome to the dedicated hub for tool creation in Unreal Engine 5. To make the learning path as clear—and actionable—as possible, this section is divided into three progressive parts, each building on the lessons of the previous one.

Part 1 — Blueprints

We start with UE5’s powerful visual‑scripting system. You’ll learn how to prototype ideas rapidly, expose parameters for designers, and iterate without recompiling. Expect hands‑on demos that get results on‑screen fast.

Video Tutorials

How to Use the BugIt Command in Unreal Engine

{% capture tutorialvideo %}Ntdtf0UY70U?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

Tool for Unreal Engine: Generate Lightmap UVs (Blueprints)

{% capture tutorialvideo %}S8XLTxFsikA?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

Tool for Unreal Engine: Cast Shadows (Blueprints)

{% capture tutorialvideo %}FwKKHfStsMg?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

Tool for Unreal Engine: Empty Actors (Blueprints)

{% capture tutorialvideo %}3-jSbxYF1W8?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

Tool for Unreal Engine: Duplicated Actors (Blueprints)

{% capture tutorialvideo %}rABhgArqbEU?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}


Part 2 — Python

With the workflow proven in Blueprints, we’ll graduate to Python scripting. Here we automate repetitive tasks, build pipeline helpers, and interface with external tools. Python lets us glue systems together and speed up daily production chores.

To trigger a Python script directly from an Editor Utility Widget, simply use the Execute Python Script node.

{% include figure image_path="/assets/images/Tools/Python/PytonExecute.png" alt="Execute Python Script node in Unreal Engine" %}

The first chapter of the Python section will focus on practicing how to clean up your Outliner and organize actors into the proper folders.
Let’s get started — here’s the video:

{% capture tutorialvideo %}3-jSbxYF1W8?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

Code:

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



Part 3 — C++

Finally, we translate our prototypes into performant, production‑ready C++. This chapter focuses on writing clean modules, extending the editor, and squeezing every drop of performance out of custom tools.

Whether you’re a technical artist looking to iterate visually, a pipeline engineer automating processes, or a programmer chasing raw speed, you’ll find something valuable in each chapter. Dive in, follow along, and let’s build better tools together!