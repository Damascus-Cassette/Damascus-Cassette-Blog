---
title: Observations on Godot's Editor for Plugins and `@tool`
image:
description:
categories:
    - Godot
    - GodotEditor
    - UI
tags:
    - ToolDev
    - UI
draft: false
uuid: dfaa6eda-3e97-4f45-b4bd-ba8880f95cf9
layout: portfolio
foam_template:
  name: Blog portfolio
  description: Blog portfolio file
  filepath: 'Damascus-Cassette-Blog/content/portfolio/2025/observations-on-godots-editor-for-plugins-and-at-tool/index.md'
date: 2025-11-24 
lastModified: 2025-11-24 
license:
---


Below is an overview of how I understand Godot and it's Editor operates. This *was* meant to be a short pre-ramble for the complex `@tool` UI article, but evolved into something much more in-depth. It's a bit loose since it's based on my experience with godot thus far, and I expect minor inaccuracies. It is the workbench after all!

I'd honestly recommend that anyone reading this to already have some experience in godot, and a tolerance for a lot of words.


---

Any Godot application's principle *structure* is a ref-counted node-tree structure, grouped and stored by prototype-tree definitions (scenes) that can contain additional information. "Singletons" or 'Autoloads' in Godot are Node/Tree instances placed under `root/Autoloads` within the process tree and made globally accessible within all scripts' namespace. Godot also heavily uses an inbuilt implementation of the observer-pattern via `Signals`. 

The Godot editor itself is running in Godot, utilizing *extensions* of the built in classes to interface with the user.

`Node`s are the foundational tree element, and can exist in a "prototype" state (partially initialized) and a fully instantiated state. Nodes are allowed to have a single script attached that, when fully initialized, is effectively (if not actually) an extension/inheritor of the node's base class. Importantly all `@export`ed variables are stored on/about the node regardless of "prototype" state. To varying degrees all base node types have functionality that executes within the game editor and the game engine, with the best example being the control/UI node's adjusting their display as they are being edited. If a Node was created from a scene, it's `.owner` attribute refers to the root node of the scene, and otherwise is null.  

`Scene`s are a blueprint of a collection of Nodes, their data (`@export`s and relationships) and associated resources. When loaded into memory they are an instance of `PackedScene`, which uses the `instantiate` function to create the tree in instances, sets `.owner`, attach resources and return it's root.

The Editor primarily changes and adjusts a scene in a "prototype" state through exposing parent-child relationships in a `Tree` in the "Scene" Panel, `@export`ed variables (& functions) in the "Inspector" panel[^1], and signals within the "Node" panel. The way the Editor handles scenes is that there is only 1 loaded scene at any given time, switching scenes dumps the old scene and loads then instantiates that the newly selected scene. Each exposed property in the inspector is an inbuilt scene instance setup with the target, the attr, and a reference to the editor autoload `EditorUndoRedoManager`. When any property is edited it registers the action directly to the `EditorUndoRedoManager`, rather than going through any intermediatory interface that registers do-undo like blender's property system.

For the godot-user's side (the app-game developer) we can utilize `@tool` in a script to allow a script to be fully instantiated & running inside the editor. This is useful *and* dangerous because it's universal. All relevant functions such as `_process()` and `_input()` are called regardless of if the scene is in the editor's 'viewport' or instantiated elsewhere in the editor, and we are left to check if `Engine.is_editor_hint() == true` to change behavior between contexts [^2]

Scenes are just dif-snapshots of node structures in memory, which can be a very powerful tool *but* one that can mix logic & data in a way that can be detrimental to most games if used as a regular save system. If using an inbuilt type to serialize data would highly recommend using an indirection of `Resources` and sub-Resources for saving game progression & inventory data, with the knowledge that loading resources from disk can *alter data on load* and namespace changes cannot be easily accounted for unless loading then converting the data.


While not entirely accurate, Godot's Editor structure is *generally* like so:


``` bash
Godot_Process
  - Resource-Pool
  - Tree-Root
    |-/ Plugins /...
    |-/ AutoLoads
    |   |-/ EditorUndoRedoManager
    |   |-/ Input
    |   |-/ ...
    |-/ Editor-Window
          |-/ InputListener
          |-/ .../CenterPanel/...
                |-/ Footer /...
                |-/ SubViewport 
                      |-/ Gizmo-Layer
                      |-/ Scene-Layer 
                            |-/ {ActiveSceneRoot *or* null} 
                                  |-/ ...
```

This can be explored through a tool like this!
https://github.com/TobiasKozel/godot_self_inspect


Regardless of the exactness of this structure, I want to point that the data being edited within godot is running within the UI in a nested ui location.
Currently the only way to access the currently loaded scene is to use special methods provided in the `EditorInspectorPlugin` class, or if a node in our scene is a tool script we can use `owner`.

For a godot tool developer most custom UI work is done through calling inbuilt functions of an `EditorInspectorPlugin` to mount your Nodes, and attaching relevant signals for scene-changes. All user-input for properties does have to implement it's own `EditorUndoRedoManager` action registration, and all other ui instances of the represented data must be set to re-draw or otherwise naturally expire and re-populate for changes

But because of Godot's flexible UI system, we do have *full access* to really mess around with things >:D

My next workbench post will be about creating a complex UI system that can handle multiple representations & relationships as declared by the Nodes being observed!


---



[^1] Generally, we can expose virtual variables with `_get_properties()->Dict`, change viewing options of variables with the `@export` family of wrappers, or override display of properties with an `EditorInspectorPlugin`. Exposed functions must have `@tool` in the script header. More on all that momentarily! 

[^2] I find myself wishing for a lot of things in developing tools for the godot editor, but a better delineation between contexts in tool scripts is certainly up there!

<!-- As a tool developer I find myself wishing 
- For a better inbuilt delination between Node instance context leading node logic
  - Ie contexts of `{Editor, EditorActiveScene, Game}`
- Better clarity on a `@tool` nodes `_notification` within the editor.
  - ie a user copying a node triggers the delete notification
- A local/default interface exposed for editing attributes
  - You have to re-create all property editors for your own UI !!!! 
- Easier contextual UI registration
- A better pattern of exposing scene information & contextual/relational information
    - Blender for instance has exposed datatypes that are displayed in different panels, while godot utilizes nested resource menus with *some* inbuilt panels for interacting with specific types. (ie shader-graph) -->
<!-- Blender's API has much more specialized data-types -->

<!-- --- Sidenote
Personally, I'm of the opinion that the editors handling of scenes does not sufficiently deliniate between data & logic state when it comes to `@tool` scripts & scenes in general. I find myself wishing for a number of things, but one of the simplist is the ability to `@tool` individual functions, and have all nodes in the editor as full instances with non-`@tool` functions un-callable and all values as constants. Something I find a bit closer to a true prototype instead of the half-prototype I currently observe. I think this could solve a lot of the UI development workarounds I have to do, such as evaluating the script class and calling static functions on 
--- End Sidenote -->

<!-- The Editor itself (and godot in general) is very focused on the barest number of moving parts, which I greatly respect, but I do wish that there was a greator 

Coming from blender I find myself wishing for a wider distinction in the editor between  -->

<!-- I have *a lot* of opinions about how I like certain UI patterns & methods in blender a lot more. -->