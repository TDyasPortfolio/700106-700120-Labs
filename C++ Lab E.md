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

Vector addition ```+```
```c++
Vector3d operator + (const Vector3d& v) const { return add(v); }
```

Vector subtraction ```-```
```c++
Vector3d operator - (const Vector3d& v) const { return subtract(v); }
```

Scalar product ```*```
```c++
double operator * (const Vector3d& v) { return dotProduct(v); }
```

Vector product ```^```
```c++
Vector3d operator ^ (const Vector3d& v) { return crossProduct(v); }
```

Vector addition ```+=```
```c++
void operator += (const Vector3d& v) {
	_x += v._x;
	_y += v._y;
	_z += v._z;
}
```

Vector subtraction ```-=```
```c++
void operator -= (const Vector3d& v) {
	_x -= v._x;
	_y -= v._y;
	_z -= v._z;
}
```

Stream out ```<<```
```c++
friend std::ostream& operator <<(std::ostream& os, const Vector3d& v) {
	os << "(" << v._x << ", " << v._y << ", " << v._z << ")";
	return os;
}
```

Stream in ```>>```
```c++
friend std::istream& operator >>(std::istream& is, Vector3d& v) {
	is >> v._x >> v._y >> v._z;
	return is;
}
```
(Assumption: This operator will only take 3 doubles with whitespace in between them as a valid input)

### Q: Add a new method to overload the unary operator ```-```

```c++
Vector3d operator - () const { return { -_x, -_y, -_z }; }
```

### Q: Use the timing code from the early labs to analyse the performance of each implementation of vector addition.

There are three ways we can perform vector addition as part of the timing payload:
- By calling the ```add``` method directly:
```c++
for (auto j = 0; j < 20; j++) {
	dummyVec = dummyVec.add(dummyVec2);
	dummyX += 1;
}
```

- By using the overridden ```+``` operator:
```c++
for (auto j = 0; j < 20; j++) {
	dummyVec = dummyVec + dummyVec2;
	dummyX += 1;
}
```

- By using the overridden ```+=``` operator:
```c++
for (auto j = 0; j < 20; j++) {
	dummyVec += dummyVec2;
	dummyX += 1;
}
```

| Loop   | Median | Mean |
|--------|--------|------|
| Method | 99     | 99   |
| +  Opp | 99     | 99   |
| += Opp | 99     | 99   |

These performance analysis results would suggest that all of these solutions perform as well as each other and compile down into the same assembly code, meaning that overriding the mathematical operators improves programmer convenience at no performance cost (because they're being inlined, and thus, replaced with the base definition by the compiler in all instances).

## Exercise 2 - Commutativity

### Q: Implement a standard method and override the ```*``` operator to multiply a vector by a single double

```c++
Vector3d multiply(const double scalar) const { return { _x * scalar, _y * scalar, _z * scalar }; }

Vector3d operator * (const double scalar) const { return multiply(scalar); }
```

### Q: Implement a multiplication of a double by a vector. Why is this more problematic than the previous specification?

In the case of ```double * Vector3d```, the left-hand operand is a built-in C++ type, and as a result it doesn't have any member functions to override to call our code. To circumvent this, we need to create a friend function which overrides the multiply operator with a double on the left and a Vector3d on the right:

```c++
friend Vector3d operator * (const double scalar, const Vector3d& v) {
	return v.multiply(scalar);
}
---------main.cpp---------
const Vector3d a(1,2,3);
const auto b = 4.1 * a;
std::cout << "b = " << b << std::endl;
```

<img width="280" height="90" alt="image" src="https://github.com/user-attachments/assets/083f76e5-9c2b-4578-825f-0ec8574bd9aa" />

## Exercise 3 - Matrices

### Q: Implement the requested functionality:

Matrix addition, with a function and an ```+``` operator override:
```c++
Matrix33d add(const Matrix33d& m) const { 
	double data[9] = {
	_row[0]._x + m._row[0]._x, _row[0]._y + m._row[0]._y, _row[0]._z + m._row[0]._z,
	_row[1]._x + m._row[1]._x, _row[1]._y + m._row[1]._y, _row[1]._z + m._row[1]._z, 
	_row[2]._x + m._row[2]._x, _row[2]._y + m._row[2]._y, _row[2]._z + m._row[2]._z
	};
	return Matrix33d(data);
}

Matrix33d operator+ (const Matrix33d& m) const {
	return add(m);
}
```

Matrix subtraction, with a function and a ```-``` operator override:
```c++
Matrix33d subtract(const Matrix33d& m) const {
	double data[9] = {
	_row[0]._x - m._row[0]._x, _row[0]._y - m._row[0]._y, _row[0]._z - m._row[0]._z,
	_row[1]._x - m._row[1]._x, _row[1]._y - m._row[1]._y, _row[1]._z - m._row[1]._z,
	_row[2]._x - m._row[2]._x, _row[2]._y - m._row[2]._y, _row[2]._z - m._row[2]._z
	};
	return Matrix33d(data);
}

Matrix33d operator- (const Matrix33d& m) const {
	return subtract(m);
}
```

Matrix multiplication with a function and a '''*''' operator override:
```c++
Matrix33d multiply(const Matrix33d& m) const {
	double a = _row[0]._x, b = _row[0]._y, c = _row[0]._z;
	double d = _row[1]._x, e = _row[1]._y, f = _row[1]._z;
	double g = _row[2]._x, h = _row[2]._y, i = _row[2]._z;
	double aa = m._row[0]._x, ab = m._row[0]._y, ac = m._row[0]._z;
	double ad = m._row[1]._x, ae = m._row[1]._y, af = m._row[1]._z;
	double ag = m._row[2]._x, ah = m._row[2]._y, ai = m._row[2]._z;

	const double margin = 1e-15;
	double data[9] = {
	a * aa + b * ad + c * ag,
	a * ab + b * ae + c * ah,
	a * ac + b * af + c * ai,
	d * aa + e * ad + f * ag,
	d * ab + e * ae + f * ah,
	d * ac + e * af + f * ai,
	g * aa + h * ad + i * ag,
	g * ab + h * ae + i * ah,
	g * ac + h * af + i * ai
	};
	for (double& v : data) {
		if (fabs(v) < margin) {
			v = 0.0;
		}
	}
	return Matrix33d(data);
}

Matrix33d operator* (const Matrix33d& m) const {
	return multiply(m);
}
```
(Because floating point multiplication when calculating the inverse later on often resulted in numbers that should be 0, but are instead on the order of ~e-16, I iterate through the data at the end and set numbers underneath a specific margin to 0.0.


Streaming in and out with friend function ```>>``` and ```<<``` operator overrides:
```c++
friend std::ostream& operator <<(std::ostream& os, const Matrix33d& v) {
	os << v._row[0]._x << " " << v._row[0]._y << " " << v._row[0]._z << std::endl
	<< v._row[1]._x << " " << v._row[1]._y << " " << v._row[1]._z << std::endl
	<< v._row[2]._x << " " << v._row[2]._y << " " << v._row[2]._z << std::endl;
	return os;
}

friend std::istream& operator >>(std::istream& is, Matrix33d& v) {
	is >> v._row[0]._x >> v._row[0]._y >> v._row[0]._z >> v._row[1]._x >> v._row[1]._y >> v._row[1]._z >> v._row[2]._x >> v._row[2]._y >> v._row[2]._z;
	return is;
}
```
(Assumption: The matrix is given as 9 doubles separated by only whitespace)

Inverse
```c++
Matrix33d inverse() const {
	double a = _row[0]._x, b = _row[0]._y, c = _row[0]._z;
	double d = _row[1]._x, e = _row[1]._y, f = _row[1]._z;
	double g = _row[2]._x, h = _row[2]._y, i = _row[2]._z;

	double det = a * (e * i - f * h) - b * (d * i - f * g) + c * (d * h - e * g);

	if (fabs(det) < 1e-12) {
		throw std::runtime_error("Determinant is 0, so the matrix doesn't have an inverse");
	}
			
	double invDet = 1.0 / det;

	double data[9] = {
		(e * i - f * h) * invDet,
		(c * h - b * i) * invDet,
		(b * f - c * e) * invDet,
		(f * g - d * i) * invDet,
		(a * i - c * g) * invDet,
		(c * d - a * f) * invDet,
		(d * h - e * g) * invDet,
		(b * g - a * h) * invDet,
		(a * e - b * d) * invDet
	};
	return Matrix33d(data);
}
```

I used the following test to determine if the matrix inverse was working correctly. Multiplying matrix A by its own inverse, if one exists, should result in an identity matrix, which it does:

```c++
int main (int, char**) {
	double a[9] = { 3, 1, 4, 1, 5, 9, 2, 6, 5 };

	Matrix33d A(a);
	Matrix33d B = A.multiply(A.inverse());

	std::cout << B;
	return 0;
}
```

<img width="98" height="130" alt="image" src="https://github.com/user-attachments/assets/f3f074f6-88e0-4e83-a3e5-fa1d466562ab" />


Transpose
```c++
Matrix33d transpose() const {
	double data[9] = {
		_row[0]._x, _row[1]._x, _row[2]._x,
		_row[0]._y, _row[1]._y, _row[2]._y,
		_row[0]._z, _row[1]._z, _row[2]._z
	};
	return Matrix33d(data);
}
```

## Exercise 4 - Vector and Matrix Multiplication

### Q: Expand the Matrix33d to allow for multiplication by a Vector3d

I wrote a multiply function in Matrix33d which takes a Vector3d as the rightmost operand and multiplies them together, returning a Vector3d as a result:

```c++
Vector3d multiply(const Vector3d& v) const {
	return Vector3d{
		_row[0]._x * v._x + _row[0]._y * v._y + _row[0]._z * v._z,
		_row[1]._x * v._x + _row[1]._y * v._y + _row[1]._z * v._z,
		_row[2]._x * v._x + _row[2]._y * v._y + _row[2]._z * v._z
	};
}

Vector3d operator*(const Vector3d& v) const {
	return multiply(v);
}
```

To test it, I wrote the following calculation:

<img width="163" height="68" alt="image" src="https://github.com/user-attachments/assets/e25b5058-e73a-489c-b3a7-b333ab83c99f" />

```c++
int main(int, char**) {
	double a[9] = { 3, 1, 4, 1, 5, 9, 2, 6, 5 };

	Matrix33d A(a);
	Vector3d v(3, 1, 4);
	Vector3d v2 = A * v;

	std::cout << v2;
	return 0;
}
```

<img width="224" height="81" alt="image" src="https://github.com/user-attachments/assets/e3188401-a0ea-4975-93b3-a1636cdf6464" />

## Exercise 5 - Internal Data Structures

### Q: Would it be advantageous to store the components of a vector in an array of rank 1, size 3, or should they be kept separate attributes?

In my opinion it would make the most sense to use an array of rank 1, size 3 to store the components of a vector. In my opinion, the most obvious advantage is that an array can be iterated over, whereas the separate named attributes of Vector3d cannot be.

### Q: Would it be advantageous to store the components of a matrix in an array of rank 2, size 3 (or rank 1, size 9), or should Matrix33d continue to use an array of Vector3d?

In my opinion, it would be much more maintainable as a programmer if Matrix33d was stored in an array of rank 2, size 3. It would condense and simplify the process of accessing a single number in the matrix: i.e. instead of ```_row[2]._y```, it could be ```matrix[2][1]```, which makes more innate sense to me and also enables us to navigate through the matrix with nested loops.

### Q: Reimplement ```Matrix33d``` with another data format and compare its performance with the timing code.

To time the pre-existing code, I created the following payload:

```c++
auto dummyX = 1.0;
double a[9] = { 3, 1, 4, 1, 5, 9, 2, 6, 5 };
Matrix33d A(a);
Matrix33d B;
-------------------------------------------
for (auto j = 0; j < 20; j++) {
	B = B * A * A.transpose();
	dummyX += 1;
}
```

Under the current situation, execution took a median of 4356 clock cycles:

<img width="365" height="85" alt="image" src="https://github.com/user-attachments/assets/a956d64f-70c4-4c7d-9df6-d254e55b9082" />

I reimplemented Matrix33d, storing the components in 3x3 array of doubles, called matrix:

```c++
class Matrix33d {
	static constexpr double _epsilon = 1.0e-8;
	double matrix[3][3]{};
```

Unfortunately, this re-implementation completed the payload in a median of 4587 clock cycles, meaning it's marginally slower than before:

<img width="345" height="79" alt="image" src="https://github.com/user-attachments/assets/02010e60-4f3a-46ee-873f-707c13e4133c" />

My theory as to why this implementation is overall slower boils down to how I reimplemented the multiplication method:

```c++
Matrix33d multiply(const Matrix33d& m) const {
	const double margin = 1e-15;
	double data[9] = {
	matrix[0][0] * m.matrix[0][0] + matrix[0][1] * m.matrix[1][0] + matrix[0][2] * m.matrix[2][0],
	matrix[0][0] * m.matrix[0][1] + matrix[0][1] * m.matrix[1][1] + matrix[0][2] * m.matrix[2][1],
	matrix[0][0] * m.matrix[0][2] + matrix[0][1] * m.matrix[1][2] + matrix[0][2] * m.matrix[2][2],
	matrix[1][0] * m.matrix[0][0] + matrix[1][1] * m.matrix[1][0] + matrix[1][2] * m.matrix[2][0],
	matrix[1][0] * m.matrix[0][1] + matrix[1][1] * m.matrix[1][1] + matrix[1][2] * m.matrix[2][1],
	matrix[1][0] * m.matrix[0][2] + matrix[1][1] * m.matrix[1][2] + matrix[1][2] * m.matrix[2][2],
	matrix[2][0] * m.matrix[0][0] + matrix[2][1] * m.matrix[1][0] + matrix[2][2] * m.matrix[2][0],
	matrix[2][0] * m.matrix[0][1] + matrix[2][1] * m.matrix[1][1] + matrix[2][2] * m.matrix[2][1],
	matrix[2][0] * m.matrix[0][2] + matrix[2][1] * m.matrix[1][2] + matrix[2][2] * m.matrix[2][2]
	};
	for (double& v : data) {
		if (fabs(v) < margin) {
			v = 0.0;
		}
	}
	return Matrix33d(data);
}
```

I think it is overall more maintainable and understandable to me than the letters-based solution, but it seems like saving all the values as local variables meant they could be accessed faster to perform the multiplication, outweighing the percieved benefits of using the rank 2, size 3 array. Still, I think that my re-implementation strikes a fairer balance between efficiency and readibility than before with its helpful nested indexing.
