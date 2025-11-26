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

For a fuller context on constraints I would recomend looking at a previous page that focuses on [some obervations abuot Godot's Editor and @tool](../observations-on-godots-editor-for-plugins-and-at-tool/index.md)

For the Nodegraph I will also note that I am purposefully not using the inbuilt `GraphUI` and `NodeUI` types, as they will be replaced soon and have some nuances in how they are used that makes things a little bit less useful for me compared to a pure UI representation.

## Content

To simplify from point A -> B, I'll outline my constraints and goals.

- A flexible UI system that can support nested and virtual data-types
- Declaration of UI views & property exposure within the originating class
- The ability to use the resulting system within both the godot editor and games
- Interactive shortcut regions & Modal property interactions
- Method to update all affected views-properties as relevant by an input action
- Application of value changes should be applied through an interface matching `UndoRedo`/`EditorUndoRedoManager`
- Exposure of availiable views and nodes within a selector of a panel
- 
<!-- - Support for parent-child relationships. -->
<!-- - No singletons for the exposure of functionality. -->

Eventually I would like to expand to allow for
- Loading data from disk directly
<!-- - Allowence for abstract relationships -->

<!-- this is all a bit much, next time I may just want to learn C++ and just fork the editor. -->

- Plugin
  - Pool
- Exposure Window
  - Panel (Pool listener)
    - Header
    - Body (Contents)

![alt text](MySystem.drawio.svg)