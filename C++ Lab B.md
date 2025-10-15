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

The median duration is the same as before at 99 clock cycles, but the mean execution time has risen from 102 to 109. This would suggest that a typical run of the payload does not change in x86 mode in comparison to x64, but that slower cases are more common when running in x86 and thus drag the mean down, even though extreme outliers are discarded when the mean is calculated.
