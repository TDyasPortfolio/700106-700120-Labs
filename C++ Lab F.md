# C++ Lab F - Gang of Three

## Exercise 1 - BigStrings

### Q: Expand the partially implemented ```BigString``` class to contain the requested functionality. Instrument your code by streaming out the current method.

#### Constructor, building a ```BigString``` taking a ```const char*``` as input:

```c++
BigString::BigString(const char* inputString)
{
	std::cout << "[Debug] BigString(const char*) called" << std::endl;
	_size = strlen(inputString);
	_arrayOfChars = new char[_size + 1];
	strcpy_s(_arrayOfChars, _size + 1, inputString);
}
```

(We add 1 to the size to account for the null terminator which ends the C-style string)

#### Copy constructor, building a deep-copied ```BigString``` taking a reference to another ```BigString``` as input:

```c++
BigString::BigString(const BigString& inputString) {
	std::cout << "[Debug] Copy constructor called" << std::endl;
	_size = inputString._size;
	_arrayOfChars = new char[_size + 1];
	strcpy_s(_arrayOfChars, _size + 1, inputString._arrayOfChars);
}
```

#### Destructor, which is automatically run to delete dynamic memory, preventing memory leaks, when the ```BigString``` goes out of scope:

```c++
BigString::~BigString() {
	std::cout << "[Debug] Destructor called" << std::endl;
	delete[] _arrayOfChars;
}
```

#### Assignment operator, which copies the size and elements of the source ```BigString``` to the target ```BigString```:

```c++
BigString& BigString::operator=(const BigString& inputString) {
	std::cout << "[Debug] Assignment operator called" << std::endl;
	if (this != &inputString) {
		delete[] _arrayOfChars;
		_size = inputString._size;
		_arrayOfChars = new char[_size + 1];
		strcpy_s(_arrayOfChars, _size + 1, inputString._arrayOfChars);
	}
	return *this;
}
```

#### Stream in and out, giving the ```_arrayOfChars``` data to an ```std::ostream``` and using an ```std::istream``` to override the current size and elements:

```c++
std::ostream& operator<<(std::ostream& sout, const BigString& inputString) {
	std::cout << "[Debug] Ostream operator called" << std::endl;
	if (inputString._arrayOfChars) { sout << inputString._arrayOfChars; }
	return sout;
}

std::istream& operator>>(std::istream& sin, BigString& inputString) {
	std::cout << "[Debug] Istream operator called" << std::endl;
	char bufferChars[inputBufferSize];
	sin >> bufferChars;
	inputString = BigString(bufferChars);
	return sin;
}
```

#### Index operators, returning a singular element of ```_arrayOfChars``` given an integer index (with both const and non-const versions)

```c++
char& operator[](int index) {
	std::cout << "[Debug] Non-const index operator called" << std::endl;
	return _arrayOfChars[index];
}

const char& operator[](int index) const {
	std::cout << "[Debug] Const index operator called" << std::endl;
	return _arrayOfChars[index];
}
```

## Exercise 2 - Test Harness

### Q: Create code to test all the functionality within BigString. Check for memory leaks using the mechanism from the lecture.

This is the memory leak-checking mechanism. It can detect if memory leaks arose as a result of memory which hasn't been properly freed by destructors:

```c++
#include <crtdbg.h>
#ifdef _DEBUG
#define new new(_NORMAL_BLOCK, __FILE__, __LINE__)
#endif

---in main()---
int flag = _CrtSetDbgFlag(_CRTDBG_REPORT_FLAG);
flag |= _CRTDBG_LEAK_CHECK_DF;
_CrtSetDbgFlag(flag);
```

#### Passing ```BigString``` to a function by value

```c++
void printByValue(BigString string) {
	std::cout << "Printed by value: " << string << std::endl;
}

---in main()---
BigString test("This is a test string");
printByValue(test);
return 0;
```

Running the function, we expect to see a copy constructor to create the parameter ```string```, an ostream operator to output ```string``` to the console, and a destructor at the end of the function when ```string``` goes out-of-scope:

<img width="626" height="206" alt="image" src="https://github.com/user-attachments/assets/ad680609-59cf-4ff1-b412-a5ea36b9cc9a" />

(We also see the constructor and destructor of the BigString in main(), which is to be expected)

#### Passing ```BigString``` to a function by reference

```c++
void printByReference(BigString& string) {
	std::cout << "Printed by reference: " << string << std::endl;
}

---in main()---
BigString test("This is a test string");
printByReference(test);
return 0;
```

Running the function, we now won't see the copy constructor and destructor from before because we're passing in a reference to a pre-existing instance of the object:

<img width="669" height="164" alt="image" src="https://github.com/user-attachments/assets/e03de898-2168-4cee-89b0-81edfc6a8703" />

#### Returning ```BigString``` from a function by value

```c++
BigString returnByValue() {
	return BigString("This is a test string");
}

---in main()---
BigString test = returnByValue();
return 0;
```

Running the function, we see only the constructor used to generate the return value, and the destructor when ```test``` goes out of scope at the end of the program.

<img width="472" height="120" alt="image" src="https://github.com/user-attachments/assets/aa42280c-e055-4023-90d8-157ec6183035" />

#### Returning ```BigString``` from a function by reference

```c++
BigString returnedByReference("This is a test string");
BigString& returnByReference() {
	return returnedByReference;
}

---in main()---
BigString& test = returnByReference();
return 0;
```

Running the function, we see the constructor and destructor running once, in relation to the global ```BigString``` (which is constructed at the start of the program and destroyed at the end). Returning and assigning the reference does not cause any extra operations to occur.

<img width="470" height="118" alt="image" src="https://github.com/user-attachments/assets/403ceb0d-4c73-44c0-9b3e-50941997e28a" />

#### Assigning one BigString object to another

```c++
BigString testOne("This is a test string");
BigString testTwo("This is a different test string");
testTwo = testOne;
```

Finally, here we should see two constuctors, an assignment operator, and then two destructors:

<img width="479" height="185" alt="image" src="https://github.com/user-attachments/assets/7e914351-8bec-4762-9b5f-1c418a42b7bd" />

#### (The program never flagged any memory leaks, because the dynamic memory being assigned is always correctly freed in the ```BigString``` destructor)

## Exercise 3 - Optimisation

### Q: Are there any situations where you think you can improve the efficency?

I could not find an obvious performance improvement in the current state of the code, but there are a couple of theories I have for possible improvements:

- Is it possible to pass the BigString by value without using the copy constructor? (I wouldn't imagine so - because you inherently need to deep copy something to pass it by value)
- I always delete and re-assign the dynamic memory in the assignment operator - could it be possible to overwrite the current memory to reduce execution time?
