# Mastering Render Targets
This repository contains the code for the companion blog post [Mastering Render Targets in Defold.](https://davabase.net/posts/mastering-render-targets-in-defold/)

Each part describes a piece of the puzzle for lining up the projection frustum to properly scale partial screen shaders.

These projects are for the [Defold](https://defold.com/) game engine, you can find more resources for Defold at the bottom.

## Part 1
Find it [here.](part_1/)
* Setting up the render pipeline.
* Creating a game object to draw to.
* Creating a new render target.
* Create a passthrough material.
* Drawing predicates to the the render target.
* Drawing the game object predicate.

## Part 2
Find it [here.](part_2/)
* Update the passthrough material to render the objects in grayscale.

## Part 3
Find it [here.](part_3/)
* Update the screen projection to only render the part of the screen within our render window.
* Add controls so we can drag the window around.

## Part 4
Find it [here.](part_4/)
* Generalize the projection math so the render window can be any aspect ratio.
* This also allows us to change the size of the render target.
* Add controls to change the scaling of the render window.

Learn more about the Defold game engine here:
https://defold.com
https://github.com/defold/defold
https://forum.defold.com

Useful references for rendering in Defold:
https://defold.com/manuals/render
https://defold.com/manuals/material
https://defold.com/manuals/shader
https://defold.com/tutorials/shadertoy