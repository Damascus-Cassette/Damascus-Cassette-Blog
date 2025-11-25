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

![alt text](MySystem.drawio.svg)