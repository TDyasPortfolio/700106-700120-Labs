# C++ Lab B

## Exercise 1 - Timing 

### Q: Try increasing the iterations of the timing program's payload from 20 to 40. Can you explain the result?

I began by running the timing program in Release x64 mode with 20 iterations (timing in Debug mode is irrelevant because the compiler optimises out pieces of code for Release mode):

```c++
for (auto j = 0; j < 20; j++) {
  dummyX = dummyX * 1.00001;
}
```

I recieved the following output, indicating a mean execution time of 102 CPU clock cycles, with a median of 99:

<img width="355" height="179" alt="image" src="https://github.com/user-attachments/assets/520fe158-e954-4dca-8241-2c6434a568a0" />

Then, changing ```j < 20``` to ```j < 40``` to execute the payload's loop 40 times, the mean rose to 201 clock cycles, with a median of 198:

<img width="352" height="169" alt="image" src="https://github.com/user-attachments/assets/77c1271d-94a3-4c2f-9e4d-0d5dbfa96371" />

#### This is because doubling the number of iterations means the CPU is doing double the calculations, and as a result it takes the CPU twice the amount of clock cycles to complete the payload.

### Q: Now try running it in both Release x64 and Release x86 mode.

I ran the initial, 20-iteration version of the payload in Release x86 mode, and got the following results:

<img width="346" height="167" alt="image" src="https://github.com/user-attachments/assets/115c9259-b43e-4822-843f-bfe5fbb3e6c9" />

#### The median duration is the same as before at 99 clock cycles, but the mean execution time has risen from 102 to 109. This would suggest that a typical run of the payload does not change in x86 mode in comparison to x64, but that slower cases are more common when running in x86 and thus drag the mean down, even though extreme outliers are discarded when the mean is calculated.

## Exercise 2 - Timing own code

### Q: Replace the payload with your own code

Like the sample payload, I edited dummy values that are later printed, to ensure that compiler does not optimise away any of the payload code:

```c++
auto dummyX = 1.0;
auto dummyY = 1.0;
-------------------------------
const auto factor = 0.34;
const auto factor2 = 2.34;
for (auto j = 0; j < 35; j++) {
	dummyX = j * cos(factor);
	dummyY += dummyX / factor2;
}
```

In Release x64 mode, it executed in a median 330 clock cycles:

<img width="354" height="171" alt="image" src="https://github.com/user-attachments/assets/dea22f0d-da67-4d4a-833e-82688b284db6" />

## Exercise 3 - Conditionals

### Q: Add the if, switch and ? conditional types to the payload to compare their performance

I'm going to use each of the conditional types to execute the following pseudocode, to ensure that each of them does exactly the same work:

```
set X to j mod 3
if X is 0, add 10 to dummyX
if X is 1, add 20 to dummyX
if X is 2, add 30 to dummyX
```

For the if payload:

```c++
for (auto j = 0; j < 20; j++) {
	auto const x = j % 3;
	if (x == 0) { dummyX += 10; }
	else if (x == 1) { dummyX += 20; }
	else { dummyX += 30; }
}
```

For the switch payload:

```c++
for (auto j = 0; j < 20; j++) {
	auto const x = j % 3;
	switch (x) {
		case 0: dummyX += 10; break;
		case 1: dummyX += 20; break;
		default: dummyX += 30; break;
	}
}
```

For the ? payload:

```c++
for (auto j = 0; j < 20; j++) {
	auto const x = j % 3;
	dummyX += (x == 0) ? 10 : ((x == 1) ? 20 : 30);
}
```

Executing and timing each payload produced the following timing data:

| Conditional | Median | Mean    |
|-------------|--------|---------|
| if          | 99     | 115.598 |
| switch      | 99     | 113.156 |
| ?           | 165    | 158.872 |

The timing program identified that the ```if``` and ```switch``` operators took roughly the same amount of CPU cycles, and the ternary operator ```?``` took noticeably longer. 

#### I think that this discrepancy shows that the ```if``` and ```switch``` operators compiled down into approximately the same assembly code, giving the same median performance with small variation in mean execution time due to random noise. I think the ternary operator ```?``` took noticeably longer because it compiled differently into assembly, and the assembly produced was less efficient for this example. 

## Exercise 4 - Branch Prediction

### Q: Demonstrate when branch prediction is working well and when it isn't

The CPU did effective branch prediction in the previous examples because they worked cyclically - as the loop goes on, x alternates between 0, 1 and 2 repeatedly, so the loop always went down the first branch, then the second branch, then the third branch. However, if we introduce an element of randomness, the branch outcomes will be essentially unpredictable, so the predictor will fail often, creating extra work and significantly slowing down the execution of the payload:

```c++
for (auto j = 0; j < 20; j++) {
	auto const x = rand() % 3;
	if (x == 0) { dummyX += 10; }
	else if (x == 1) { dummyX += 20; }
	else { dummyX += 30; }
}
```

Compared to the previous ```if``` example, the execution was much slower as expected, because of the significantly lower rate of accurate branch predictions (and the small amount of overhead associated with the random number generator):

| Branch Prediction | Median | Mean    |
|-------------------|--------|---------|
| Accurate          | 99     | 115.598 |
| Random            | 2277   | 2281.32 |

## Exercise 5 - Nested Loops

### Q: Compare the 4 different methods of exiting a nested loop.

This is the pseudocode they will be executing:

```
loops = 100
for j in 0 -> loops
	for k in 0 -> loops
		if j * k = 1533 (j = 21, k = 73)
			add 1 to dummyX and exit loop 
```

Method 1 - Each condition section of the loops has two conditions, for the loop control and the exit condition:

```c++
const auto loops = 100;
bool found = false;
for (auto j = 0; j < loops && !found; j++) {
	for (auto k = 0; k < loops && !found; k++) {
		if (j * k == 1533) {
			dummyX += 1;
			found = true;
		}
	}
}
```

Method 2 - An additional if statement immediately after the inner loop to catch and propogate a break statement:

```c++
const auto loops = 100;
bool found = false;
for (auto j = 0; j < loops; j++) {
	for (auto k = 0; k < loops; k++) {
		if (j * k == 1533) {
			dummyX += 1;
			found = true;
			break;
		}
	}
	if (found) break;
}
```

Method 3 - A goto statement in the inner loop:

```c++
const auto loops = 100;
for (auto j = 0; j < loops; j++) {
	for (auto k = 0; k < loops; k++) {
		if (j * k == 1533) {
			dummyX += 1;
			goto end_of_loops;
		}
	}
}
end_of_loops:
```

Method 4 - A return statement in a lambda function:

```c++
const auto loops = 100;
auto lambdaLoop = [&]() {
	for (auto j = 0; j < loops; j++) {
		for (auto k = 0; k < loops; k++) {
			if (j * k == 1533) {
				dummyX += 1;
				return true;
			}
		}
	}
	return false;
};
auto dummyY = lambdaLoop();
```

Using these payloads, I obtained the following timing data:

| Exit Method | Median | Mean    |
|-------------|--------|---------|
| Method 1    | 2343   | 2342.43 |
| Method 2    | 4323   | 4342.69 |
| Method 3    | 4323   | 4336.65 |
| Method 4    | 4323   | 4351.70 |

#### The results tells us that methods 2, 3 and 4 have all compiled into basically the same assembly, hence their identical medians and approximately equal means. In my opinion, this tells us that in method 1, the compiler has noticed that the conditions on both loops will immediately be broken if ```found``` is ever set to True, and has optimised around it. The results suggest that you should seek to use method 1, because it is the fastest of the 4.

## Exercise 6 - Range-based loops

### Q: How do range-based loops compare in performance to standard loops?

This is the pseudocode that they will be executing:

```
values = 100-long vector of ints containing "8"
loop through and add every element of values to dummyX
```

As a standard loop:

```c++
const auto loops = 100;
std::vector<int> values(loops, 8);
for (auto j = 0; j < loops; j++) {
	dummyX += values[j];
}
```

As a range-based loop:

```c++
const auto loops = 100;
std::vector<int> values(loops, 8);
for (auto value : values) {
	dummyX += value;
}
```

This gave the following timing data:

| Loop        | Median | Mean    |
|-------------|--------|---------|
| Standard    | 693    | 711.060 |
| Range-based | 693    | 712.691 |

#### My expectation was that the range-based loop was going to take longer, because values need to be copied over from the original vector in order to used in the range-based loop. It is clear from the timing results, that these code snippets have each compiled into equivalent assembly, so my expectations have not been met. I think this is because the payload being executed was too simple and perhaps the compiler was able to take note of and optimise the use of the range-based loop.

## Exercise 7 - Architecture

To compare x64 architecture to x86 architecture, I'm going to re-do the conditional and nested loop exercises, producing x86 code:

| Conditional | Median | Mean    |
|-------------|--------|---------|
| if          | 99     | 115.598 |
| switch      | 99     | 113.156 |
| ?           | 165    | 158.872 |
| if (x86)    | 396    | 405.464 |
| switch (x86)| 396    | 404.697 |
| ? (x86)     | 165    | 173.805 |

| Exit Method | Median | Mean    |
|-------------|--------|---------|
| Method 1    | 2343   | 2342.43 |
| Method 2    | 4323   | 4342.69 |
| Method 3    | 4323   | 4336.65 |
| Method 4    | 4323   | 4351.70 |
| Method 1 (x86  | 4125   | 4277.01 |
| Method 2 (x86) | 7854   | 8385.54 |
| Method 3 (x86) | 7854   | 8545.22 |
| Method 4 (x86) | 7854   | 9851.99 |

#### With the exception of the ternary conditional operator ```?```, everything executed significantly slower when running in x86 mode. In my view, this is because the fundamental characteristics of x86 (i.e. access to fewer registers, which necessitate more regular cache access) mean that particular fundamental actions simply take more CPU cycles to be fully executed in an x86 context, thus explaining the across-the-board execution penalty for using x86.
