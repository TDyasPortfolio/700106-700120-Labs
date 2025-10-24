# Vulkan Lab B - Procedural Geometry and Model Loading

## Exercise 1 - Create a flat grid

I use nested loops in the createGrid function to generate vertices for the grid's intersections, and then use another set of nested loops to generate indices, turning the vertices into a grid of triangles using triangle strips with the primitive restart value 0xFFFF:

<img width="1155" height="519" alt="image" src="https://github.com/user-attachments/assets/d7027553-dacf-436a-8279-fcbfdf3517a7" />

<img width="804" height="634" alt="image" src="https://github.com/user-attachments/assets/5c1e6e7d-b2a3-4d27-92b9-3eb0b9f0247f" />

## Exercise 2 - Create a wavy terrain

I edited the createGrid function to set a z co-ordinate based on the x and y co-ords:

<img width="819" height="55" alt="image" src="https://github.com/user-attachments/assets/b17734da-4f6f-4a6c-9066-84e242657d5d" />

This created elevation changes in the grid:

<img width="806" height="642" alt="image" src="https://github.com/user-attachments/assets/058b4a89-50bc-48f1-88fd-8289fccdcc1e" />

## Exercise 3 - Procedurally generate a cylinder



## Exercise 4 - Create a resuable geometry utility


## Exercise 5 - Load external models with Assimp
