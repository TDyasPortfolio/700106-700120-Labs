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

I wrote the following function to generate a cylinder, which:
 - Generates 2 circles in a for loop, each composed of ```int resolution``` number of vertices
 - Generates the vertices for the centers of the top and bottom caps
 - Uses triangle strips to create the caps with the top and bottom vertices
 - Uses triangle strips to draw the walls between the top and bottom vertices

```c++
void createCylinder(float radius, float height, int resolution, std::vector<Vertex>& outVertices, std::vector<uint16_t>& outIndices) {
    const auto twoPi = 4 * acos(0.0);
    for (auto w = 0; w < resolution; w++) {
        Vertex vt, vb;
        vt.pos = glm::vec3((radius * cos((twoPi / resolution) * w)), (radius * sin((twoPi / resolution) * w)), height);
        vb.pos = glm::vec3((radius * cos((twoPi / resolution) * w)), (radius * sin((twoPi / resolution) * w)), 0);
        vb.color = vt.color = glm::vec3(1);
        outVertices.push_back(vt);
        outVertices.push_back(vb);
    }
    Vertex centerTop(glm::vec3(0, 0, height), glm::vec3(1));
    outVertices.push_back(centerTop);
    auto topCenterIndex = outVertices.size() - 1;
    Vertex centerBottom(glm::vec3(0, 0, 0), glm::vec3(1));
    outVertices.push_back(centerBottom);
    auto bottomCenterIndex = outVertices.size() - 1;
    for (auto w = 0; w < resolution; w++) {
        auto next = (w + 1) % resolution;
        outIndices.push_back(w * 2);
        outIndices.push_back(next * 2);
        outIndices.push_back(topCenterIndex);
        outIndices.push_back(0xFFFF);
        outIndices.push_back(w * 2 + 1);
        outIndices.push_back(next * 2 + 1);
        outIndices.push_back(bottomCenterIndex);
        outIndices.push_back(0xFFFF);
    }
    for (auto w = 0; w <= resolution; w++) {
        auto next = (w % resolution) * 2;
        auto bottom = next + 1;
        outIndices.push_back(next);
        outIndices.push_back(bottom);
    }
}

void loadModel() {
    createCylinder(0.5, 0.5, 20, vertices, indices);
}
```

<img width="792" height="632" alt="image" src="https://github.com/user-attachments/assets/1bf7e097-aea6-4b97-830a-289ecbdff8b4" />


## Exercise 4 - Create a resuable geometry utility

I created the class MeshHelper with grid and cylinder functions which can cumulatively add meshes to the same vertices and indices vectors as before to render multiple objects:

<img width="901" height="350" alt="image" src="https://github.com/user-attachments/assets/be943d5e-fa68-4b6c-9d4d-272a7838face" />

<img width="791" height="638" alt="image" src="https://github.com/user-attachments/assets/bf42c3c0-b3e4-4466-9ee0-c392d8658933" />

## Exercise 5 - Load external models with Assimp

I wrote a function in my MeshHelper class which can import an OBJ model from a filename and add its vertices and indices to the vectors to be rendered:

<img width="1166" height="395" alt="image" src="https://github.com/user-attachments/assets/073cf467-8f22-478a-88b0-32a6a9c2f554" />

<img width="806" height="633" alt="image" src="https://github.com/user-attachments/assets/41668dac-29c3-4ad1-a155-f162da59e149" />

