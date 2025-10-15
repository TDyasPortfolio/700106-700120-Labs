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

