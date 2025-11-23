---
title: "Godot @Tool: Custom Nodegraph & UI System"
image:
description:
categories:
    - Godot
    - NodeGraph
tags:
    - Tooling
    - "@Tool"
draft: true
uuid: e71dc445-22dd-4c7d-bc9c-d722e5db46e3
layout: workbench
foam_template:
  name: Blog workbench
  description: Blog workbench file
  filepath: 'Damascus-Cassette-Blog/content/workbench/2025/Custom-Tool-UI-System-and-Nodegraph-In-Godot/index.md'
date: 2025-11-23 
lastModified: 2025-11-23 
license:
comments: true
---

## Preramble:

This article will cover a lot of the structure and thoughts so far on creating a custom nodegraph as part of an `EditorInspectorPlugin` in Godot. The purpose and allowences I place here will become more clear in future articles, as here I will be focusing exclusivly on the difficulties and eases of creating a custom tool UI System within godot. 

Within UI & Nodegraphs I'm very coloured by Blender's Addon & UI System, which is comperable to QT in some aspects. Thus it's been interesting to both miss it and revel in the freedom without it!


# Content

Hello! 

Godot is a fantastic game engine that due to it's self-referencial nature can be a very interesting canidate for any standalone complex 2d-3d software project. Godot's Editor runs within Godot, and as such it allows for some very interesting tricks and allowences, such as loading any scene into the Godot UI itself or inspecting & editing a game at runtime (the primary difference between these is target location in the program and some debugging tools attached as far as I can tell!).

## Godot's Editor Structure & `@tool` Behavior (Generally)

Principally, all of Godot's editor and Artifacts are composed of an Node-Instance tree structure that is used to track & access primary relationships (Parent; Children), and objects not ascociated with the root (Object, RefCounted, Resources, ect). "Singletons" in godot are just nodes/node-trees instanced into a pre-defined location of "root/autoloads/" and are attached to the globally accessable namespace for easy reference from within code. 


### Nodes 

Nodes are 

### Scenes & `@tool`

Godot's scenes on disk can be best thought of as blueprints & flags for a tree of nodes & list of inner-resources, where the contents recorded are a dif of the original class & information changed each node's `@export`ed variables[^1]. 

<!-- As you can nest scenes within the editor, a foreign scene's node's value overrides are applied ontop of the the original data (though I'm currently uncertain of timing in relation to gdscript instantiation). -->

A script is a primary variable of almost any object in godot, and allows a script-class to extend the functionality of a node instance at runtime. Importantly, `@export`ed vars declared by the script are not stored on an instance of the script-class but the Node-instance itself. Unless `@tool` is included in the head of a gdscript file, the gdscript-class is not fully instanced within the editor's active scene and cannot execute *non-static* functions. When `@tool` is included in the head of the gdscript file, it is instanced fully in both the editor's viewport structure *and* everywhere else it may be instantiated. Non-`@tool` scripts are *never* fully instanciated in the editor tree, no matter if the editor location is in the parent-chain or not. Representations of a tree are stored as a PackedScene, but within the editor instantiated to their tree. 

I wish this was not the case as it makes custom tooling somewhat more difficult, since all scripts require @tool and you may not want some to run as you are editing their structure.

While not an accurate representation I find it helpful to visualize the editor as this tree: 

```
Process
    - Root/
        /EditorAutoloads
            /EditorUndoRedoManager
        /Autoloads
            /...
        /Plugins  
            /...
        /EditorWindow/
            /Input/...
            /UI/
                /Panels/
                    /Main Panel/...
                        /Scene-Window/...
                            /.../{ActiveSceneRoot-Prototype}/...
                        /Footer/...
    - Editor Data ?
        - [Objects, Refcounted, GdScripts, Ect]
        - [Nodes]
    - Edited Scene Data ?
        - [Objects, Refcounted, GdScripts, Ect]
        - [Nodes]
```

<!-- 
### Scripting 

Within GDscript, all editor-exposed classes must be their own script file & declared with the `class_name ...` at the top. An implication of this
Godot enforces OOP principles strictly through single class inheritance (extension), dis-allowences of changing variables in child classes and asserting a uniform interface for each function. Thus in effect all children match the same static interface as their parent, but can have extended behavior.
 Class names are also declared globaly, but are mostly and easy reference to the script's type/ -->

[^1] 
Generally, there are options to change the viewing and writing of variables, the `@export` family of flags has a lot of options. For this post I'll use `@export` to refer to all wrappers in the export family that I'm aware of.