# C++ Lab E - Vectors and Matrices

## Exercise 1 - Basic Vectors

### Q: Why are some methods in the class ```Vector3d``` inlined and some of them not?

Though they do not use the keyword inline explicitly, there are multiple numerical operators that are declared directly in Vector3d's class definition, meaning that they are implicity inlined. They are inlined because they are smaller, performance-critical functions that are called often - so inlining them, suggesting to the compiler that the function call can be substituted with its definition, ultimately eliminiates the overhead of performing explicit function calls, especially with common maths operations such as addition.

### Q: Implement the requested functionality.

Vector product (cross product)

```c++
Vector3d crossProduct(const Vector3d& v) { return { _y * v._z - _z * v._y, _z* v._x - _x * v._z, _x* v._y - _y * v._x,}; }
```

Scalar product (dot product)

```c++
double dotProduct(const Vector3d& v) { return _x * v._x + _y * v._y + _z * v._z; }
```

### Q: Add new methods to overload the following binary operators.

vector addition ```+```
```c++
Vector3d operator + (const Vector3d& v) const { return add(v); }
```

vector subtraction ```-```
```c++
Vector3d operator - (const Vector3d& v) const { return subtract(v); }
```

scalar product ```*```
```c++
double operator * (const Vector3d& v) { return dotProduct(v); }
```

vector product ```^```
```c++
Vector3d operator ^ (const Vector3d& v) { return crossProduct(v); }
```

vector addition ```+=```
```c++
void operator += (const Vector3d& v) {
	_x += v._x;
	_y += v._y;
	_z += v._z;
}
```

vector subtraction ```-=```
```c++
	void operator -= (const Vector3d& v) {
		_x -= v._x;
		_y -= v._y;
		_z -= v._z;
	}
```

stream out ```<<```
```c++

```

stream in ```>>```
```c++

```

### Q: Add a new method to overload the unary operator ```-```

```c++
Vector3d operator - () const { return { -_x, -_y, -_z }; }
```

## Exercise 2 - Commutativity

## Exercise 3 - Matrices

## Exercise 4 - Vector and Matrix Multiplication

## Exercise 5 - Internal Data Structures
