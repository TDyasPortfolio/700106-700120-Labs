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
