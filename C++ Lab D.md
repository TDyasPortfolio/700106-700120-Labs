# C++ Lab D - Object Parser and Arrays

## Exercise 1 + 2 - Object Parser

### Q: Complete the parser to successfully parse ```sample.obj``` and use the data to populate the vertex, texture, normal and face arrays, and output the result.

I began with an approach which ignored object names, but could successfully parse the sample file and output the data of its one object. I also had to write:
- a string splitting function so that I could parse the face data (because the 3 numbers are not delimited by whitespace but instead slashes, i.e. ```f 1/1/1 5/2/1 7/3/1 3/4/1```), because one does not exist by default in C++
- a function which parsed the data from the vectors and concatenated everything back into a string which is printed and also outputted in ```output.txt```

```c++
while (fin >> tag) {
	if (tag == "o") {
		continue;
	}
	else if (tag == "v") {
		Vertex v;
		fin >> v.x >> v.y >> v.z;
		vertices.push_back(v);
	}
	else if (tag == "vt") {
		Texture t;
		fin >> t.u >> t.v;
		textures.push_back(t);
	}
	else if (tag == "vn") {
		Normal n;
		fin >> n.x >> n.y >> n.z;
		normals.push_back(n);
	}
	else if (tag == "f") {
		std::string tokens[4];
		fin >> tokens[0] >> tokens[1] >> tokens[2] >> tokens[3];
		for (auto token : tokens) {
			std::vector<std::string> numbers = split(token, '/');
			vertexIndices.push_back(stoi(numbers[0]));
			textureIndices.push_back(stoi(numbers[1]));
			normalIndices.push_back(stoi(numbers[2]));
		}
	}
	else {
		continue;
	}
}
```

```c++
std::vector<std::string> split(const std::string& str, char delimiter) {
	std::vector<std::string> tokens;
	size_t start = 0;
	size_t end = str.find(delimiter);
	while (end != std::string::npos) {
		tokens.push_back(str.substr(start, end - start));
		start = end + 1;
		end = str.find(delimiter, start);
	}
	tokens.push_back(str.substr(start));
	return tokens;
}
```

```c++
std::string printAll(std::vector<Vertex>& vertices, std::vector<Texture>& textures, std::vector<Normal>& normals, std::vector<int>& vertexIndices, std::vector<int>& textureIndices, std::vector<int>& normalIndices) {
	std::string output;
	output += "Vertices: (";
	for (auto vertex : vertices) {
		output += " [" + std::to_string(vertex.x) + ", " + std::to_string(vertex.y) + ", " + std::to_string(vertex.z) + "] ";
	}
	output += ")\n\nTextures: (";
	for (auto texture : textures) {
		output += " [" + std::to_string(texture.u) + ", " + std::to_string(texture.v) + "] ";
	}
	output += ")\n\nNormals: (";
	for (auto normal : normals) {
		output += " [" + std::to_string(normal.x) + ", " + std::to_string(normal.y) + ", " + std::to_string(normal.z) + "] ";
	}
	output += ")\n\nFaces: (";
	for (auto i = 0; i < vertexIndices.size() / 4; i++) {
		output += " [" + std::to_string(vertexIndices[i * 4]) + "/" + std::to_string(textureIndices[i + 4]) + "/" + std::to_string(normalIndices[i * 4]) + " " + std::to_string(vertexIndices[i * 4 + 1]) + "/" + std::to_string(textureIndices[i + 4 + 1]) + "/" + std::to_string(normalIndices[i * 4 + 1]) + " " + std::to_string(vertexIndices[i * 4 + 2]) + "/" + std::to_string(textureIndices[i + 4 + 2]) + "/" + std::to_string(normalIndices[i * 4 + 2]) + " " + std::to_string(vertexIndices[i * 4 + 3]) + "/" + std::to_string(textureIndices[i + 4 + 3]) + "/" + std::to_string(normalIndices[i * 4 + 3]) + "] ";
	}
	output += ")";
	return output;
}
```

<img width="1471" height="355" alt="image" src="https://github.com/user-attachments/assets/e4074f5b-9e66-49f2-b146-8e66eb3ea817" />

### Q: Expand the project to allow multiple objects to be parsed and successfully parse ```cubes.obj```

In order to parse the data for multiple objects, I created an Object struct, which holds its name and all the data that will be input about it:

```c++
struct Object {
	std::string name;
	std::vector<Vertex> vertices;
	std::vector<Texture> textures;
	std::vector<Normal> normals;
	std::vector<int> vertexIndices;
	std::vector<int> textureIndices;
	std::vector<int> normalIndices;
};
```

I changed the parsing logic so that, when the tag says "o", suggesting the end of one object and the beginning of a new one, the instance of ```Object``` that's currently being added to is pushed back into the ```std::vector<Object>``` and then wiped clean for the next object's input data. I also made sure that the current object is pushed into the vector when the input is finished, which is then followed by the output to the console and the text file. In addition to this, I made a simple new function to iterate through the objects and convert them to strings, so that they can be output separately:

```c++
std::vector<Object> objects;
Object currentObject;
std::string tag;
while (fin >> tag) {
	if (tag == "o") {
		if (!currentObject.name.empty()) {
			objects.push_back(currentObject);
			currentObject = Object();
		}
		fin >> currentObject.name;
	}
	else if (tag == "v") {
		Vertex v;
		fin >> v.x >> v.y >> v.z;
		currentObject.vertices.push_back(v);
	}
	else if (tag == "vt") {
		Texture t;
		fin >> t.u >> t.v;
		currentObject.textures.push_back(t);
	}
	else if (tag == "vn") {
		Normal n;
		fin >> n.x >> n.y >> n.z;
		currentObject.normals.push_back(n);
	}
	else if (tag == "f") {
		std::string tokens[4];
		fin >> tokens[0] >> tokens[1] >> tokens[2] >> tokens[3];
		for (auto token : tokens) {
			std::vector<std::string> numbers = split(token, '/');
			currentObject.vertexIndices.push_back(stoi(numbers[0]));
			currentObject.textureIndices.push_back(stoi(numbers[1]));
			currentObject.normalIndices.push_back(stoi(numbers[2]));
		}
	}
	else {
		continue;
	}
}
objects.push_back(currentObject);
std::string results = printObjects(objects);
std::cout << results;
fout << results;
return 0;
```

```c++
std::string printObjects(std::vector<Object> objects) {
	std::string output;
	for (auto object : objects) {
		output += "Object Name: " + object.name + "\n" + printAll(object.vertices, object.textures, object.normals, object.vertexIndices, object.textureIndices, object.normalIndices) + "\n\n";
	}
	return output;
}
```

This program successfully parses the two objects in ```cubes.obj``` and is able to output ```Cube``` and ```Cube.001``` separately:

<img width="1899" height="555" alt="image" src="https://github.com/user-attachments/assets/ae93a73a-faaa-4c19-93f7-8eb84ff200e6" />

## Exercise 3 - Tuples

### Q: Wrap your object parser within a new function, which takes only the file name as a parameter, and returns a tuple containing the size of the largest object and the level of the largest object.

I got quite confused by the terminology used in this exercise, and I could not find any explanations in the taught material, so there are a number of assumptions I made in order to implement this idea:

- The **largest** object is the one with the most vertices
- Thus, returning the size of this object means returning the number of vertices
- I did not know what the **level** of the object refers to, so I took it to mean "the number of **faces**"

As a result, I wrote a function which parses the text file, keeping track of the number of vertices/faces in the current object, and also the number of vertices and faces in the largest object, outputting the larget as part of a tuple. It returns a value of (0, 0) if there is an error opening the file:

```c++
std::tuple<int, int> tupleParser(std::string filename) {
	std::ifstream fin(filename);
	if (!fin) {
		return std::make_tuple(0, 0);
	}

	auto currentVertices = 0; 
	auto currentFaces = 0;
	auto bestVertices = 0;
	auto bestFaces = 0;
	auto currentBiggest = false;

	std::vector<Object> objects;
	Object currentObject;
	std::string tag;
	while (fin >> tag) {
		if (tag == "o") {
			if (!currentObject.name.empty()) {
				currentBiggest = false;
				objects.push_back(currentObject);
				currentObject = Object();
				currentVertices = 0;
				currentFaces = 0;
			}
			fin >> currentObject.name;
		}
		else if (tag == "v") {
			Vertex v;
			fin >> v.x >> v.y >> v.z;
			currentObject.vertices.push_back(v);
			currentVertices++;
			if (currentVertices > bestVertices) {
				currentBiggest = true;
			}
		}
		else if (tag == "vt") {
			Texture t;
			fin >> t.u >> t.v;
			currentObject.textures.push_back(t);
		}
		else if (tag == "vn") {
			Normal n;
			fin >> n.x >> n.y >> n.z;
			currentObject.normals.push_back(n);
		}
		else if (tag == "f") {
			std::string tokens[4];
			fin >> tokens[0] >> tokens[1] >> tokens[2] >> tokens[3];
			for (auto token : tokens) {
				std::vector<std::string> numbers = split(token, '/');
				currentObject.vertexIndices.push_back(stoi(numbers[0]));
				currentObject.textureIndices.push_back(stoi(numbers[1]));
				currentObject.normalIndices.push_back(stoi(numbers[2]));
			}
			currentFaces++;
		}
		else {
			continue;
		}
		if (currentBiggest) {
			bestVertices = currentVertices;
			bestFaces = currentFaces;
		}
	}
	return std::make_tuple(bestVertices, bestFaces);
}

int main(int argc, char** argv) {

	const auto results = tupleParser(argv[1]);
	std::cout << "(" + std::to_string(std::get<0>(results)) + ", " + std::to_string(std::get<1>(results)) + ")";

}
```

To test this out, I made a dummy OBJ file with 3 objects:
- One with 4 vertices and 4 faces, like a triangle-base pyramid
- One with 5 vertices and 5 faces, like a square-base pyramid
- One with 8 vertices and 6 faces, like a cube

<img width="332" height="783" alt="image" src="https://github.com/user-attachments/assets/9e2238dd-4328-478f-ae7c-095bc252b678" />

The cube is the expected largest object, so the output should be (8, 6), which it was:

<img width="391" height="84" alt="image" src="https://github.com/user-attachments/assets/586dc01d-68df-425b-bf7e-a1c858065b90" />

## Exercise 4 - Span and Arrays

### Q: Reimplement the program using the array template, which wraps a vanilla C array within a C++11 template.

I don't have the pass the size of ```listOfValues``` in a separate parameter anymore, but I do still need to define the size of the array as part of the function's signature, i.e. ```std::array<int, size_t>``` was not acceptible as a valid signature for it:

```c++
int findLargestValueV1(std::array<int, 100> listOfValues)

std::array<int, numOfValues> listOfValues;
```

<img width="351" height="100" alt="image" src="https://github.com/user-attachments/assets/c3b68e04-cdf6-413c-ad24-a05460954e04" />

### Q: Reimplement it again to use C++20's ```span``` to pass the array into the function

```std::span``` is a reference type with a size that does not have to be known at runtime. As a result, you can simply pass ```std::span<const int>``` to the function and it works exactly the same way as before without having to pass in the size of the array into the function:

```c++
int findLargestValueV1(std::span<const int> listOfValues)
```

<img width="319" height="90" alt="image" src="https://github.com/user-attachments/assets/dea681bc-e3fd-4c00-88e2-44b15b36ee6a" />
