# C++ Lab A - Basics of C++

## Exercise 1 - Copilot Tutorial 

I set up the required example program with a breakpoint:

<img width="703" height="392" alt="image" src="https://github.com/user-attachments/assets/a79a57d2-0c9d-407e-a10b-07193e4c98f3" />

I was able to successfully ask Copilot why the value of ```args``` at that point in the code was ```string[0]```:

<img width="910" height="482" alt="image" src="https://github.com/user-attachments/assets/d153ff26-679b-4da0-b120-b7556f6ce42a" />

Asking a follow-up question, Copilot suggested that I use a try-catch block to catch cases where args has no command-line arguments:

<img width="839" height="691" alt="image" src="https://github.com/user-attachments/assets/595ceaa8-fbaa-46c0-8558-c881b6b76279" />

Then, stepping forward and getting the exception from the incorrect line of code, it analysed and troubleshooted the exception and gave a solution:

<img width="877" height="893" alt="image" src="https://github.com/user-attachments/assets/c2d98dca-3fa1-4efe-8612-ce1cc42ce84b" />

Afterwards, I set a conditional breakpoint that only actives when ```item == "Fred"```:

<img width="1026" height="262" alt="image" src="https://github.com/user-attachments/assets/a4bd74cc-3c6d-4662-b7cd-ef0897c52224" />

Finally, I tested that the conditional breakpoint works with command-line arguments "5 John Fred Joe":

<img width="504" height="202" alt="image" src="https://github.com/user-attachments/assets/727560e8-317e-48d5-ab9f-663e680e9133" />

And when the breakpoint activates, we can see that the current value of ```item``` is "Fred":

<img width="627" height="139" alt="image" src="https://github.com/user-attachments/assets/1930f620-c5bc-4313-ad40-74567487fdeb" />

## Exercise 2 - Hello World

The Hello World program linked, compiled and ran successfully in Debug mode...

<img width="621" height="344" alt="image" src="https://github.com/user-attachments/assets/6cb69576-e1cd-4855-bca8-2297ff9075db" />

...and then again in Release mode:

<img width="1711" height="183" alt="image" src="https://github.com/user-attachments/assets/29f7358d-1e73-4ff0-84a7-72e85c595c2c" />

## Exercise 3 - Console Window

By requesting an integer value from the user, we delay the closing of the console window until after the value has been input:

<img width="564" height="258" alt="image" src="https://github.com/user-attachments/assets/f7997cbf-9fd2-4db6-93e5-862ec337a6a9" />

<img width="476" height="209" alt="image" src="https://github.com/user-attachments/assets/a18c747a-c8b6-42f3-b474-aeda7115cb9c" />

## Exercise 4 - Includes

If we compile the program without the line ```#include <iostream>```, we get compiler errors from undeclared functions relating to the IO stream:

<img width="582" height="248" alt="image" src="https://github.com/user-attachments/assets/5ada09b3-bdb3-49d1-a432-1214022811e6" />

## Exercise 5 - Namespace

If we tell the program we are ```using namespace std```, nothing changes, but it does allow us to remove ```std::``` from functions and the program will still work correctly, because the ```using namespace``` declaration tells the compiler where to look for names and labels it cannot find by default:

<img width="435" height="314" alt="image" src="https://github.com/user-attachments/assets/983981fd-b8c8-4bf8-ada8-c8d284a7cf4b" />

<img width="542" height="190" alt="image" src="https://github.com/user-attachments/assets/7e53e1f1-6471-4189-979d-7493d40fe226" />

## Exercise 6 - Creating a new project

I created an empty C++ project called Temperature in a different folder.

<img width="365" height="255" alt="image" src="https://github.com/user-attachments/assets/81de7a46-5183-451c-939b-589cee69fd1e" />

## Exercise 7 - Temperature

This program takes the Fahrenheit temperature and outputs it as Celsius:

<img width="723" height="292" alt="image" src="https://github.com/user-attachments/assets/2b09f26b-f389-4f1b-bd63-53f484cb4d25" />

<img width="524" height="190" alt="image" src="https://github.com/user-attachments/assets/cbe08b40-9ae9-4119-90c8-c881d9e800b5" />

<img width="534" height="143" alt="image" src="https://github.com/user-attachments/assets/f0d7cf35-a0d2-43d8-8bfc-6c9b40717f2e" />

## Exercise 8 - Auto, const and casting

I added the auto keyword, designated constants wherever possible, and explicitly casted to a type when it was important:

<img width="858" height="316" alt="image" src="https://github.com/user-attachments/assets/4b86c475-9c06-4a07-9acf-fbaeda599096" />

This makes the code overall easier to understand because:

- ```auto``` means you don't need to worry about what type something is in the initialiser, because the compiler can figure it out
- ```const``` ensures you can't accidentally edit something that you need to keep safe
- Explicit casts reduce bugs caused by accidentally casting something to an erroneous type implicitly

## Exercise 9 - Static Assert

When the program is set to x64, the static assert that pointers and ints are the same size does not pass:

<img width="654" height="159" alt="image" src="https://github.com/user-attachments/assets/b33a166a-fc81-43a9-b2d0-d5f9dd4c0870" />

However, the same thing does not happen when the architecture is set to x86 - the program compiles and runs with no errors:

<img width="818" height="555" alt="image" src="https://github.com/user-attachments/assets/04b2bba7-9d9a-42e2-a747-8626d1a28b35" />

In x86, we can assert that both ```sizeOfInt``` and ```sizeOfPointer``` are independantly equal to 4, and both asserts will pass:

<img width="908" height="514" alt="image" src="https://github.com/user-attachments/assets/de98ae7b-48ed-4b43-803f-a86ab5bf900c" />

This is because x86 is a 32-bit architecture, so you need 32 bits, therefore 4 bytes, to point anywhere you want. If we repeat this in x64, the assertion that pointers are only 4 bytes will not pass because the pointers have to be 8 bytes long to address everywhere they need to, when using the 64-bit architecture:

<img width="632" height="176" alt="image" src="https://github.com/user-attachments/assets/d5f3572e-910c-4885-b007-d2312136f7a5" />

If we instead assert that ```sizeOfInt == 4``` and ```sizeOfPointer == 8```, the assertions pass in x64:

<img width="911" height="484" alt="image" src="https://github.com/user-attachments/assets/a9f4904e-7b49-4953-a352-5912248c693e" />

## Reflection

This lab session has refamiliarised me with some of the fundamentals of programming in C++, as well as informing my future debugging process with help from Copilot Chat

