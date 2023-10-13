# Mastering Render Targets in Defold: Part 1

In this part we create a render target, draw all objects to it, and then sample the target in a quad material.

We create a new render target in the init function:
```lua
-- Create a new render target.
local color_params = {
    format = render.FORMAT_RGBA,
    width = render.get_width(),
    height = render.get_height()
}
local target_params = {[render.BUFFER_COLOR_BIT] = color_params }
self.target = render.render_target("target", target_params)
```
We set the width and height of the target to the resolution of the game and use standard settings for storing color data.

In the update function we add the code to draw something to the render target, in our case we'll draw anything with the "tile" predicate.
We do this after all game objects are drawn, just before the GUI is drawn.
```lua
-- Set the render target to our target.
render.set_render_target(self.target)
-- Clear the render target with the games clear color.
render.clear({
    [render.BUFFER_COLOR_BIT] = self.clear_color,
    [render.BUFFER_DEPTH_BIT] = 1,
    [render.BUFFER_STENCIL_BIT] = 0
})
-- Draw all objects with the "tile" tag in their material.
render.draw(self.tile_pred, {frustum = frustum})
-- Reset the render target.
render.set_render_target(render.RENDER_TARGET_DEFAULT)
```
Since we do this with the normal camera projection, everything visible by the camera will be drawn to our render target.

We create a new game object file, we'll call it "window" since it will look like a little window on the screen.

Give it a model component and assign the built in `quad.dae` as the model.

We'll also create a new material, called `window.material` and assign it to the model.

We'll assign the builtin `sprite.vp` as the vertex program in the material, since we're not going to modify it.

And we'll create a new fragment program called `window.fp`.
The fragment shader will just draw the texture from the sampler directly:
```glsl
varying mediump vec2 var_texcoord0;

uniform lowp sampler2D texture_sampler;

void main()
{
	gl_FragColor = texture2D(texture_sampler, var_texcoord0.xy);
}
```

We have to make sure we add a `view_proj` vertex constant.

We also need to add a texture sampler to set the render target to.
We call it `texture_sampler` because that is what is specified in the fragment shader:

Finally we add a tag to specify when this material will be drawn.
We don't want to use "tile" because that will cause a loop. Let's call it "window."

We add a new render predicate to the init function of our render script using the "window" tag we specified in the material:
```lua
self.window_pred = render.predicate({"window"})
```

Finally in the update function of the render script we draw our quad.
We do this just after we draw to the render target:
```lua
-- Set the texture in slot 0 of the material to our render target.
render.enable_texture(0, self.target, render.BUFFER_COLOR_BIT)
-- Draw the window quad.
render.draw(self.window_pred)
-- Unset the texture slot.
render.disable_texture(0, self.target)
```

Now if we add the `window.go` game object to our main collection we can see it rendered on the screen.
The builtin `quad.dae` is 1 pixel by 1 pixel, so we can scale it up to something that can be seen and move it near the center of the screen.

It's important that the quad have the same aspect ratio as the game otherwise the rendered texture will be squashed or stretched. We'll change that fact later.