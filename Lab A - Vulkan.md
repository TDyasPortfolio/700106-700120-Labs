# Vulkan Lab A - Simple Shapes

## Exercise 1 - Draw two triangles without using vertex indices

The goal of this exercise is to render 2 triangles, instead of the quad which comes up on screen. This begins by locating the vector containing the quad's vertices:

<img width="511" height="149" alt="image" src="https://github.com/user-attachments/assets/744d281b-ec00-4ef9-8392-2df77ca4e356" />

Then, I expanded the vertex vector to include 6 vertices, representing 2 separate triangles:

<img width="538" height="189" alt="image" src="https://github.com/user-attachments/assets/453cf600-d640-4829-a73e-8405fa962a76" />

I do not have to change the way the vertex buffer is created in ```createVertexBuffer()``` because it already works off of ```vertices.size()```, which we have just changed the value of by adding 2 new vertices:

<img width="810" height="54" alt="image" src="https://github.com/user-attachments/assets/9e6e1eca-1074-4b1c-84f0-53e55c426a37" />

Finally, in ```recordCommandBuffer()```, I replace the call for ```vkCmdDrawIndexed()``` with a call for ```vkCmdDraw()```, to draw the primitive non-indexed vertices. We make it draw the correct number of vertices by calling ```vertices.size()```:

<img width="1038" height="83" alt="image" src="https://github.com/user-attachments/assets/4639055f-b694-415b-891a-8bd725972e10" />

Now, instead of the single quad, the program outputs 2 triangles rotating in the same manner as before:

<img width="784" height="629" alt="image" src="https://github.com/user-attachments/assets/12124d59-ce5c-4210-af06-2637ce98c30b" />

## Exercise 2 - Draw two squares using an index buffer

The goal of this exercise is to render 2 squares, composed of two triangles apiece, using an index buffer to reuse the vertices where they overlap.

```
0 ---- 1
|    / |
|  /   |
2 ---- 3
```

In the above example, by indexing the points, we only need to declare 1 and 2 once, even though they're used in both the 012 and 123 triangles.

We start by declaring the 8 points we're going to use for those 4 triangles:

<img width="543" height="242" alt="image" src="https://github.com/user-attachments/assets/fbc42c47-7b77-4a67-ad43-9e27f5d2243d" />

Then, we index the points to turn them into 4 triangles, whilst reusing the ones that overlap. The order ensures a consistent winding pattern, so we don't show any back faces to the camera:

<img width="525" height="84" alt="image" src="https://github.com/user-attachments/assets/05670542-d1de-4115-ac8a-83c167f7764c" />

A staging buffer is used to copy the index data to the GPU in ```createIndexBuffer()```:

<img width="769" height="397" alt="image" src="https://github.com/user-attachments/assets/56df7785-cd53-4b90-a066-f6c32c68d38c" />

Then in ```recordCommandBuffer```, we use ```vkCmdDrawIndexed``` to draw with the index buffer, after binding it with ```vkCmdBindIndexBuffer```.

<img width="883" height="86" alt="image" src="https://github.com/user-attachments/assets/413b80b8-5e52-47e7-a734-062fd7181027" />

The result is that we have successfully displayed the two squares, with 4 triangles total:

<img width="794" height="600" alt="image" src="https://github.com/user-attachments/assets/63a98a38-bbcf-42fa-a4e5-fa08921a9e4c" />

# Exercise 3 - Draw the four walls of a cube

We now need to declare the 8 vertices of a cube:

<img width="528" height="247" alt="image" src="https://github.com/user-attachments/assets/1eb8dd4a-6785-4f53-9bd5-284521e51261" />

In order to make 8 triangles, to draw the 4 walls of the cube, we need to make 24 vertex indices.

<img width="841" height="84" alt="image" src="https://github.com/user-attachments/assets/b0aeb5ae-7120-4662-9f9d-ce1d7dbd7e60" />

As the buffers access the size of the ```vertices``` and ```indices``` vectors, we do not need to worry about editing the buffer sizes.

We do need to disable backface culling in the graphics pipeline so we can see all sides of the cube through the hole in the top:

<img width="440" height="42" alt="image" src="https://github.com/user-attachments/assets/d1414af3-0684-44cf-9db1-26dbae44b2e4" />

<img width="534" height="486" alt="image" src="https://github.com/user-attachments/assets/92351d69-326d-44f6-8d3c-ff8c047fe613" />

# Exercise 4 - Wireframe Rendering

If we change the rasterizer's ```polygoneMode``` to ```VK_POLYGON_MODE_LINE```, it will show only the edges of the cube.

<img width="515" height="42" alt="image" src="https://github.com/user-attachments/assets/f472292a-b4ad-4234-b720-f5e7dc2b4653" />

<img width="550" height="483" alt="image" src="https://github.com/user-attachments/assets/4f434241-da49-4bc0-aa7a-a907ca5cbd3e" />

# Exercise 5 - Render the cube's vertices as points

If we change the graphics pipeline's topology member from ```VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST``` to ```VK_PRIMITIVE_TOPOLOGY_POINT_LIST```, we can render each vertex of the cube as a point:

For visibility purposes, all the vertices have been changed to white:

<img width="494" height="482" alt="image" src="https://github.com/user-attachments/assets/69af6fab-d73d-46aa-bc8a-635248d10e5c" />

# Exercise 6 - Render the edges of the cube as lines

This requires a new set of indices, which define the edges of the cube as 12 lines of 2 points each:

<img width="841" height="94" alt="image" src="https://github.com/user-attachments/assets/1fc89624-9117-484a-93ef-2c1b45ff26fc" />

It also requires the graphics pipeline topology to be set to ```VK_PRIMITIVE_TOPOLOGY_LINE_LIST```:

<img width="610" height="50" alt="image" src="https://github.com/user-attachments/assets/2ea97260-5148-41b7-94a8-13941f1f4de1" />

<img width="456" height="444" alt="image" src="https://github.com/user-attachments/assets/013ea19f-ce67-453e-91db-44ac42f356d1" />

# Exercise 7 - Triangle Strips

A triangle strip is a more efficient way to render a series of connected triangles where each subsequent vertex connects with the previous 2 to form the next triangle. This sometimes requires we create degenerate triangles, which have no area, to restart the strip in a new location.

For this, we need a new index buffer - when the same index is repeated, we've created a degenerate triangle which won't be rendered because it has no area. Instead of 24 indices, we only need 15:

<img width="546" height="91" alt="image" src="https://github.com/user-attachments/assets/41849b4c-fe7a-4226-bfe0-8e7731cb1640" />

We also need to change the graphic pipeline's topology to ```VK_PRIMITIVE_TOPOLOGY_TRIANGLE_STRIP```:

<img width="676" height="57" alt="image" src="https://github.com/user-attachments/assets/d11f624f-697d-4d6a-9a76-a5e1453ec235" />

The cube now renders in one efficient draw call:

<img width="445" height="469" alt="image" src="https://github.com/user-attachments/assets/43f089c8-3132-4dc6-93d2-c058089ec3dd" />


