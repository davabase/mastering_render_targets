# Mastering Render Targets in Defold: Part 4

In this part we will make it so that no matter what the shape of the render quad for our window is, the projection of the objects behind it will be correct. We also add additional controls to change the scaling of the window game object.

We update the projection code in the `update` function of `pipeline.render_script` to compensate for the different size of the render target:
```lua
-- Get the width and height of the window and the render target.
local box_width = self.window_box.right - self.window_box.left
local box_height = self.window_box.top - self.window_box.bottom
local render_target_width = render.get_render_target_width(self.target, render.BUFFER_COLOR_BIT)
local render_target_height = render.get_render_target_height(self.target, render.BUFFER_COLOR_BIT)

-- Calculate the zoom compensation for the window.
local zoom_x = render_target_width / box_width
local zoom_y = render_target_height / box_height
local projected_width = render.get_window_width() / zoom_x
local projected_height = render.get_window_height() / zoom_y
local zoomed_proj = vmath.matrix4_orthographic(self.window_box.left,
                                               self.window_box.left + projected_width,
                                               self.window_box.bottom,
                                               self.window_box.bottom + projected_height,
                                               self.near, self.far)

-- Set the projection.
render.set_projection(zoomed_proj)
```
By taking the ratio between the size of the render target and the size of the quad, we can calculate the compensation needed for the projection bounding box.

We update the `on_input` function of our `window.script` to look like this:
```lua
if action.x and action.y then
    -- Save the mouse position as a vector.
    self.mouse_position = vmath.vector3(action.x, action.y, 0)

    -- Check if we are close to the edges to select them for scaling.
    local scale_distance = 6
    self.can_scale_left = (math.abs(action.x - self.left) < scale_distance and
                            action.y > self.bottom - scale_distance and
                            action.y < self.top + scale_distance) or self.scaling_left
    self.can_scale_right = (math.abs(action.x - self.right) < scale_distance and
                            action.y > self.bottom - scale_distance and
                            action.y < self.top + scale_distance) or self.scaling_right
    self.can_scale_bottom = (math.abs(action.y - self.bottom) < scale_distance and
                                action.x > self.left - scale_distance and
                                action.x < self.right + scale_distance) or self.scaling_bottom
    self.can_scale_top = (math.abs(action.y - self.top) < scale_distance and
                            action.x > self.left - scale_distance and
                            action.x < self.right + scale_distance) or self.scaling_top

    -- Check if we have clicked.
    if action_id == hash("mouse_button_left") then
        if action.pressed then
            -- Set scaling if we are close to an edge.
            if self.can_scale_left then
                self.scaling_left = true
            elseif self.can_scale_right then
                self.scaling_right = true
            end
            if self.can_scale_bottom then
                self.scaling_bottom = true
            elseif self.can_scale_top then
                self.scaling_top = true
            end
            -- Set dragging if we are not scaling and we are inside the window.
            if not self.scaling_left and not self.scaling_right and not self.scaling_bottom and not self.scaling_top and
            (action.x > self.left and action.x < self.right and action.y > self.bottom and action.y < self.top) then
                -- If we are inside the window calculate the offset and set dragging.
                self.offset = self.mouse_position - go.get_position()
                self.dragging = true
            end
        elseif action.released then
            -- If we have released the mouse unset dragging and scaling.
            self.dragging = false
            self.scaling_left = false
            self.scaling_right = false
            self.scaling_bottom = false
            self.scaling_top = false
        end
    end
end
```
Here we add four sets of two new variables, `can_scale` and `scaling`.
We check if the mouse is close enough to each of the edges on that edge's axis and that the mouse within the window on the other axis.

Then we set the scaling flag if we have clicked the mouse. We don't want to enable dragging if we are scaling so we add a check for that too.
If we have released the mouse, all the flags are reset.

Now we can use these flags to control the scale of the game object in the `update` function:
```lua
-- Scale.
local scale = go.get_scale()
if self.scaling_left then
    local scale_offset = self.mouse_position.x - self.left
    go.set_scale(vmath.vector3(math.max(1, scale.x - scale_offset), scale.y, 1))
elseif self.scaling_right then
    local scale_offset = self.mouse_position.x - self.right
    go.set_scale(vmath.vector3(math.max(1, scale.x + scale_offset), scale.y, 1))
end

scale = go.get_scale()
if self.scaling_bottom then
    local scale_offset = self.mouse_position.y - self.bottom
    go.set_scale(vmath.vector3(scale.x, math.max(1, scale.y - scale_offset), 1))
elseif self.scaling_top then
    local scale_offset = self.mouse_position.y - self.top
    go.set_scale(vmath.vector3(scale.x, math.max(1, scale.y + scale_offset), 1))
end
```
We set the scale of the game object based on where the mouse has dragged the edges.
We need to make sure the scale doesn't get too small or become negative so we use a max with 1 on all the scale values.

Then we can update our line drawing code so that the lines change color if we are hovering over them:
```lua
-- Draw lines around the window with a 1 pixel border.
    local red = vmath.vector4(1, 0, 0, 1)
    local blue = vmath.vector4(0, 0, 1, 1)
    local left_color = red
    local right_color = red
    local bottom_color = red
    local top_color = red
    if self.can_scale_left then left_color = blue end
    if self.can_scale_right then right_color = blue end
    if self.can_scale_bottom then bottom_color = blue end
    if self.can_scale_top then top_color = blue end
    msg.post("@render:", "draw_line", {
        start_point = vmath.vector3(self.left - 1, self.bottom - 1, 0),
        end_point = vmath.vector3(self.left - 1, self.top + 1, 0),
        color = left_color })
    msg.post("@render:", "draw_line", {
        start_point = vmath.vector3(self.left - 1, self.top + 1, 0),
        end_point = vmath.vector3(self.right + 1, self.top + 1, 0),
        color = top_color })
    msg.post("@render:", "draw_line", {
        start_point = vmath.vector3(self.right + 1, self.top + 1, 0),
        end_point = vmath.vector3(self.right + 1, self.bottom - 1, 0),
        color = right_color })
    msg.post("@render:", "draw_line", {
        start_point = vmath.vector3(self.right + 1, self.bottom - 1, 0),
        end_point = vmath.vector3(self.left - 1, self.bottom - 1, 0),
        color = bottom_color })
```

Finally we want to resize the render target if the shape of the window has changed.
In `pipeline.render_script` we update the `on_message` function to check for changing window size and change the render target size:
```lua
elseif message_id == hash("update_window_box") then
    self.window_box = message

    -- Update the size of the render target if the size of the window changed.
    if render.get_render_target_width(self.target, render.BUFFER_COLOR_BIT) ~= self.window_box.right - self.window_box.left or
    render.get_render_target_height(self.target, render.BUFFER_COLOR_BIT) ~= self.window_box.top - self.window_box.bottom then
        render.set_render_target_size(self.target,
                                      self.window_box.right - self.window_box.left,
                                      self.window_box.top - self.window_box.bottom)
    end
end
```
