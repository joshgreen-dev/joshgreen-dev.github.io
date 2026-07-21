---
layout: post
title: "The workflow that finally stopped my prints looking 3D-printed"
date: 2026-07-21
image: /images/geo-upscale/layerlines-watercolour.jpg
description: "I stopped trying to sand layer lines off my prints and started hiding them with surface texture instead. Here is the exact workflow I use now, and why it beats sanding on most models."
tags: [3d printing, layer lines, stl, texture, workflow]
keywords: "hide 3d print layer lines, add texture to stl, stl upscaler, layer lines without sanding"
---

![A 3DBenchy shown half with rough print layer lines and half smooth and textured, with a simple one-click tool on one side and a tangle of Blender nodes on the other](/images/geo-upscale/layerlines-watercolour.jpg)

I have a shelf in my workshop I think of as the graveyard. It is where the almost-good prints live. Clean geometry, right colour, perfect fit, and one problem: you can see every layer line the moment the light hits them. For a long time my answer to that shelf was sandpaper. This post is about why I stopped sanding and what I do instead.

## What nobody tells you about sanding

Sanding works. On a flat panel, with an afternoon spare, you can get a smooth injection-moulded look. I am not arguing against it.

But I print a lot of organic shapes. Figures, handles, terrain, props. On those, sanding is a slow way to wreck the detail you printed in the first place. Every pass with the grit knocks the sharp edges down. You start with a crisp model and finish with a softer, slightly rounded version of it that happens to be smoother. After a few of those I started asking a different question. Not how do I remove these ridges, but why are they even visible.

## Layer lines only show up because the surface is plain

This is the whole idea, and once it clicked I could not unsee it.

A layer line is a regular, repeating feature. Fine horizontal ridges, evenly spaced, on a smooth surface. Your eye is very good at spotting a regular pattern against a plain background, and that contrast is what reads as 3D printed. The lines are not ugly by themselves. They are ugly because they have nothing to hide behind.

So there are two ways out. Remove the ridges, which is sanding, slow, and hard on detail. Or remove the plain surface, by giving the whole thing a texture of its own. Wood grain, stone, stipple, woven cloth, any irregular relief. Once the surface is busy everywhere, the layer stepping stops standing out, because now it competes with a pattern instead of sitting alone on a blank wall.

That is why a stone-textured planter looks finished off the bed, while the same planter printed smooth looks 3D printed no matter how well you tuned the machine.

## The workflow I actually use

The old catch was how you add that texture. The proper answer is a displacement or remesh modifier in Blender, and I want to be fair about that route because I tried it. Blender is free and powerful. It is also one of the steeper learning curves in software. Getting a clean displacement onto an STL means learning remeshing, UVs, texture coordinates and modifier stacks, which is a real evening or two the first time, not a quick job. And you cannot easily hand it to someone else, because driving Blender headless from a script is its own little project.

So the workflow I use now is short:

1. Take the STL I already have, smooth surfaces and all.
2. Run it through ModelDirectory's browser-based [STL upscaler](https://modeldirectory.org/upscale-stl/), which bakes real surface relief into the mesh geometry.
3. Slice and print as normal.

Because the texture becomes actual geometry, not a painted-on image and not a material setting, it survives slicing and prints on any filament through any slicer. There is nothing to install and no account, which is the part that got it into my regular routine instead of my someday pile.

## A few things I got wrong at first

- Match the texture depth to your layer height. If the relief is shallower than one layer, the lines can still peek through on glossy filament. A slightly deeper grain fixes it.
- Organic textures are the most forgiving. Wood, leather and stone hide ridges completely, because irregular relief beats regular relief every time. Save the fine geometric stipples for grips and panels.
- You are hiding the lines, not removing them. If you need a mirror-smooth lens or a display piece that has to look manufactured, texture is the wrong tool. Sand and polish, or use a chemical smoothing pass. For most other prints, texture wins on both time and looks.

## The graveyard shelf is empty now

Not because I got better at sanding. Because I stopped trying to. The prints that used to land there now get a minute in the browser before they ever reach the slicer, and they come off the bed looking like objects instead of prototypes.

Layer lines were never a defect I had to grind away. They were a regular pattern that stops being visible the moment the surface has a more interesting one to compete with. Add the texture first, and the printer does the rest.
