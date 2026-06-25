---
layout: post
title: "Microsoft Is Killing 3D Viewer on July 1. Here's What I'm Using Instead."
date: 2026-05-28
tags: [3dprinting, windows, tools]
description: "Microsoft is permanently removing 3D Viewer from the Microsoft Store on July 1, 2026. As someone who works with STL and OBJ files daily, here's what I switched to."
---

If you work with 3D files on Windows, you need to know this: Microsoft is permanently removing 3D Viewer from the Microsoft Store on **July 1, 2026**. Not deprecating it quietly. Removing it. You won't be able to download or reinstall it after that date.

This is the last domino in Microsoft's retreat from 3D. Paint 3D was removed in November 2024. 3D Builder is gone. Windows Mixed Reality is dead. The entire "Creators Update" era from 2017 is officially over.

## Why This Matters If You 3D Print

If you work with STL files from Thingiverse, Printables, or your own designs, 3D Viewer was the default quick-look tool on Windows. Double-click an STL, see what it looks like, decide if it's worth slicing. Simple. Fast. Built-in.

Microsoft's suggested replacement is Babylon.js Sandbox, a browser-based viewer. It doesn't support STL files. The most common format in 3D printing, and their official alternative can't open it.

## What I Switched To

I've been using [GeometryViewer](https://geometryviewer.com) for the past few months. Browser-based, handles everything I throw at it.

It opens STL, OBJ, GLB, GLTF, 3MF, FBX, PLY, and STEP. Drag and drop, no install. It's a PWA so you can install it once and it works offline. Measurement tools, cross-sections, material previews. You can share models via URL, which is genuinely useful when someone on Discord asks "does this model look right to you?"

The thing that matters most for printing: it handles STL files with proper normals and gives you a realistic material preview. I can see what a print will actually look like before I slice it.

## The Full Timeline of Microsoft's 3D Exit

For context, here's how we got here:

| Product | Status |
|---------|--------|
| Remix 3D (model sharing) | Shut down |
| Windows Mixed Reality | Deprecated Dec 2023, removed in Win11 24H2 |
| HoloLens 2 | Production stopped Oct 2024, end of life 2028 |
| Paint 3D | Removed from Store November 4, 2024 |
| 3D Builder | Removed |
| **3D Viewer** | **Removed from Store July 1, 2026** |

Every single component of the 2017 "3D for Everyone" initiative is now dead.

## What About Existing Installs?

If you already have 3D Viewer installed, it won't be auto-deleted. It'll keep working. But you won't get security patches, and if you do a clean Windows install or get a new PC, you can't reinstall it.

There was also a serious security vulnerability in 3D Viewer's FBX parser: CVE-2024-20677, remote code execution, CVSS score 7.8. Microsoft's fix was to permanently disable FBX support rather than patch the bug. That should tell you everything about how much investment this app was still getting.

## My Recommendation

Don't wait for July 1. Switch now and get used to a new workflow before the deadline. For 3D printing specifically, [GeometryViewer](https://geometryviewer.com) is the closest thing to what 3D Viewer did, and it works in any browser and supports more formats.

If you need something heavier for textures, rigging, or animation, Blender is obviously the answer. But that's overkill for previewing an STL before printing.

The era of Microsoft caring about 3D on the desktop is over. Browser-based tools have gotten good enough that we don't need them to.
