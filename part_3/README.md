# Mastering Render Targets in Defold: Part 3

In this part we will change the screen projection so that we only draw what's behind the window. We'll also add some interactivity so we can drag the window around.

We create a new script called `window.script` and add it to our `window.go` game object.
In the script we calculate the bounding box in the `init` function:
```lua
function init(self)
    local position = go.get_position()
    local scale = go.get_scale()
    self.left = position.x - scale.x / 2
    self.right = position.x + scale.x / 2
    self.bottom = position.y - scale.y / 2
    self.top = position.y + scale.y / 2

    msg.post('@render:', 'update_window_box', {
        left = self.left,
        right = self.right,
        bottom = self.bottom,
        top = self.top
    })
end
```
Since Defold game objects scale from the center we just need to add or subtract half the scale to find the edges.
We also have to tell the render script these parameters, we do this by sending a message just after the calculations.

In `pipeline.render_script` we need to receive the message in the `on_message` function.
We'll tag on our new message id condition to the end:
```lua
elseif message_id == hash("update_window_box") then
    self.window_box = message
end
```

We should also initialize `self.window_box` in the `init` function, just underneath our target setup code:
```lua
self.window_box = { left = 0, right = 1, bottom = 0, top = 1 }
```

Just before we enable the `target` texture and draw to it, we want to set the projection to be the same as our window.
This will only draw the things that are within this small window:
```lua
-- Set the projection to the window box.
local window_proj = vmath.matrix4_orthographic(self.window_box.left,
                                               self.window_box.right,
                                               self.window_box.bottom,
                                               self.window_box.top,
                                               self.near, self.far)
render.set_projection(window_proj)
```
It's important that we reset the projection to the default camera projection after drawing to the render target, otherwise our window will be drawn with the wrong projection:
```lua
-- Reset the projection.
render.set_projection(proj)
```
We can reorganize the code in the render script a little bit so that we only set this projection once.

We'll draw four lines on the borders of the window in the `update` function of `window.script`.
If we're going to be updating the position we'll need to recalculate and send the bounding box as well:
```lua
function update(self, dt)
    -- Calculate the bounding box.
    local position = go.get_position()
    local scale = go.get_scale()
    self.left = position.x - scale.x / 2
    self.right = position.x + scale.x / 2
    self.bottom = position.y - scale.y / 2
    self.top = position.y + scale.y / 2

    -- Send the parameters to the render script.
    msg.post('@render:', 'update_window_box', {
        left = self.left,
        right = self.right,
        bottom = self.bottom,
        top = self.top
    })

    -- Draw red lines around the window with a 1 pixel border.
    local red = vmath.vector4(1, 0, 0, 1)
    msg.post("@render:", "draw_line", {
        start_point = vmath.vector3(self.left - 1, self.bottom - 1, 0),
        end_point = vmath.vector3(self.left - 1, self.top + 1, 0),
        color = red })
    msg.post("@render:", "draw_line", {
        start_point = vmath.vector3(self.left - 1, self.top + 1, 0),
        end_point = vmath.vector3(self.right + 1, self.top + 1, 0),
        color = red })
    msg.post("@render:", "draw_line", {
        start_point = vmath.vector3(self.right + 1, self.top + 1, 0),
        end_point = vmath.vector3(self.right + 1, self.bottom - 1, 0),
        color = red })
    msg.post("@render:", "draw_line", {
        start_point = vmath.vector3(self.right + 1, self.bottom - 1, 0),
        end_point = vmath.vector3(self.left - 1, self.bottom - 1, 0),
        color = red })
end
```
In the input bindings in `game.input_binding` we'll add a new mouse left button input called `mouse_button_left`:

Now we can acquire input focus in the `init` function of `window.script`:
```lua
-- Enable input.
msg.post(".", "acquire_input_focus")
```

In the `on_input` function we can see if the mouse is within the window when clicked and calculate the offset from the center of the game object:
```lua
function on_input(self, action_id, action)
	-- Check if we have clicked inside the window.
	if action_id == hash("mouse_button_left") then
		-- Save the mouse position as a vector.
		self.mouse_position = vmath.vector3(action.x, action.y, 0)
		if action.pressed then
			if action.x > self.left and action.x < self.right and action.y > self.bottom and action.y < self.top then
				-- If we are inside the window calculate the offset and set dragging.
				self.offset = self.mouse_position - go.get_position()
				self.dragging = true
			end
		elseif action.released then
			-- If we have released the mouse unset dragging.
			self.dragging = false
		end
	end
end
```
Read more about mouse input from the [Defold docs.](https://defold.com/ref/stable/go/#on_input)

Now in our `update` function we can move the game object if dragging is set:
```lua
-- Drag.
if self.dragging then
    go.set_position(self.mouse_position - self.offset)
end
```
We do this before re-calculating the bounding box positions.
We'll also want to initialize `self.dragging` and `self.mouse_position` in the `init` function.