# AVR Math speed
We all know AVRs are no speed demons, but exactly how fast or slow are they?

These numbers were calculated with a sketch that uses the event system to count system clocks only between the two specific points in the sketch that the math occurs at - takeOverTCA0(), pick an unconnected pin, set it output, and connect it to an event channel generator, and set TCA0 as user of that event channel. Then configure TCA0 to count while event input A is high. digitalWriteFast(pin,HIGH) will start the time in 1 clock cycle, digitalWriteFast(pin,LOW) will stop it so you can read the value. This is an extremely powerful technique for code execution time measurement!

In the case of integer operations other than division, the speed it constant regardless of the data.
In the case of all floating point operations and all division, the runtime depends on the values of the two variables.

This is very small except for the case of the 64-bit value. To give forcasts more likely to be pessimistic than optimistic, for 64-bit integers, uses the product of two random longs for the first operand (numerator) and a 64-bit datatype holding a 16-bit random value as the second (the denominator).

Obviously, integer division can be done very quickly if you notice that the denominator is larger than the numerator - the result is simply 0. ) For all others, random 32=bit numbers are used and truncated to fit the variable, if appropriate this means int8 and int16 get some positive and some negative numbers. This does not make a big different to speed it turns out. With 64-bit integers, even that simple case (where the result is zero) was found to take under 400 clock cycles, whereas in the more demanding (and, in our judgement, more realistic) tests it took more like 1750 clock cycles.

Interestingly, floating point division is often faster than dividing two random longs. This sounds absurd, until you consideer how floats are stored. They have a sign bit, which is obviously straightforward to work with, and a 23-bit mantissa that gets division applied to it, followed be normalization and adjustment of the significand, which is simpler. So we'd expect it to take about what a hypothetical

I don't quite understand why the algorithm for 8-bit division runs in the same time as 16-bit division.

But comparing 16 bit int with a long, we see that it increases by a factor of 2.57. Thus it should not surprise that the runtime for a floating point number (which requires dividing a 23 bit number then doing some comparatively easy normalization) takes about 2x as long as a 16-bit integer, but as not long as a 32-bit integer

## Moving to 64-bit datatypes changed two things dramatically:
First, the division runtime was influenced much more strongly by the values being divided. As in - numbers lower than 400 are seen if the numerator is smaller than the denominator, and if one simply uses 64-bit datatypes to hold random 32-bit values, the times are very similar to 32-bit division. But the smaller the denominator relative to the numerator, the slower the division. With two random 32-bit numbers multiplied to get each 64-bit value, the times varied widly from under 400 to over 1000 clocks, and the numbers shown below (1500-2000 clocks) reflect division by a random int16 - basically, the message is that if you have to divide large datatypes you can't even guess at the execution time without having some idea about the relative magnitude of the values.  And that 64 bit math is agonizingly slow, and that division of floats and longs is a also pretty damned slow.

Numbers were generated on a tiny3226, and interrupts were disabled while doing all math. The time taken to write the value to memory was subtracted from these numbers, so these should represent just the time the math takes. Where min and max are not given, the time is constant. Allow for several clocks error due to overhead


## Execution time in clocks; minimum maximum and average
**WARNING - these were generated on a modern AVR with hardware multiply (AVRxt). Classic AVRs of all sorts, if they have to store values to main memory, and are doing so with a pointer, will have more overhead (1 clock per byte) Results for multiply and divide will be slower for classic tinyAVRs (AVRe).

| Datatype      |   Add |   Sub | Mult. | Divide |
|---------------|-------|-------|-------|--------|
| Min (float)   |    97 |    99 |   133 |    460 |
| Mean (float)  | 112.9 | 118.8 | 141.1 |  478.9 |
| Max (float)   |   141 |   151 |   151 |    499 |
| Min (int8)    |     9 |     5 |     8 |    221 |
| Mean (int8)   |     5 |     5 |     8 |    233 |
| Max (int8)    |     9 |     5 |     8 |    243 |
| Min (int16)   |    14 |    14 |    22 |    225 |
| Mean (int16)  |    14 |    14 |    22 |  235.1 |
| Max (int16)   |    14 |    14 |    22 |    247 |
| Min (int32)   |    20 |    20 |    81 |    603 |
| Mean (int32)  |    20 |    20 |    81 |  604.4 |
| Max (int32)   |    20 |    28 |    81 |    609 |
| Min (int64)   |    54 |    54 |   321 |   1461 |
| Mean (int64)  |    54 |    54 |   321 | 1734.4 |
| Max (int64)   |    54 |    54 |   321 |   1999 |
| Est. (float64)|   302 |   316 |   560 |   1500 |

### Main takeaways

1. The big takeaway here is that division is agonizingly slow (we all knew that), especially on large data types. It's slower for integer types (effect for floats smaller) when the denominator is smaller, sometimes dramatically.
2. All math on floats is slow as hell - they take about 5.5 times as long as integer operations on the equally sized long - except for division, 
3. And so (we all knew this) you should try to avoid using floating point numbers as well as division, [see this document for some general techniquess](Fast_Math_Tricks.md)
4. int64's are mind bogglingly slow, I suspect this is because there aren't enough working register
5. There is a reason that we don't have 64 bit floating point datatypes, and why people avoid 64-bit integers whenever possible.
The extrapolated values for float64's are based on observed patterns, not actual tests. There is no float65 datatype. As you can see, the performance would be truly abysmal.
6. Never use an uint64 to store 64 flags. The performance impact is profound. 

## How do I get a float64? 
You don't. Look at the performace numbers. The estimates for float64's, which don't exist, are very crude: I multiplied the float speed by the ratio of the int64 time to the int32 time (and in the case of division, I think it would scale worse than that), are intended as an answer to questions about "why don't we have 64bit floats?!" (1. no avr-gcc support until v10, and 2. Performance would be unspeakably bad.)

## Appendix: Approximate speed in us at 16, 20, 24, 32, and 48 MHz
These can serve as a bit of a sanity check when you're designing algorithms for AVR that might be getting carried away with math. If you're running a tiny at MHz, and you have an algorithm that calls arduino's Random function 600 times (each of which requires a 32-bit division) that's 22.8 ms. If those random values were being used to control the colors of a string of 200 neopixels from one "frame" to the next, and you want 30FPS, (33ms per iteration), you know you're going to be CPU-bound (you're spending 2/3rds of the time you have just generatign random numbers - much less using them, and you also have to output the data, which takes non-negligible time too). I happened to already know that that examople was out of reach as long as Arduino's random was involved from experience; by implementing a more efficient xor-shift algorithm, a dramatic speedup was enabled, making it possible to meet the 30fps target and extend the length of the string.

| type  | A/S@16 | Mul@16 | Div@16 | A/S@20 | Mul@20 | Div@20 | A/S@24 | Mul@24 | Div@24 | A/S@32 | Mul@32 | Div@32 | A/S@48 | Mul@48 | Div@48 |
|-------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|
| float |   7.24 |  8.82  |  29.93 |   5.79 |   7.05 |  23.95 |   4.83 |   5.88 |  19.95 |   3.62 |   4.41 |  14.97 |   2.41 |   2.94 |   9.98 |
| int8  |   0.31 |  0.50  |  14.56 |   0.25 |   0.40 |  11.65 |   0.21 |   0.33 |   9.71 |   0.16 |   0.25 |   7.28 |   0.10 |   0.17 |   4.85 |
| int16 |   0.88 |  1.38  |  14.69 |   0.70 |   1.10 |  11.75 |   0.58 |   0.92 |   9.80 |   0.44 |   0.69 |   7.35 |   0.29 |   0.46 |   4.90 |
| int32 |   1.25 |  5.06  |  37.78 |   1.00 |   4.05 |  30.22 |   0.83 |   3.38 |  25.18 |   0.63 |   2.53 |  18.89 |   0.42 |   1.69 |  12.59 |
| int64 |   3.38 | 20.06  | 108.40 |   2.70 |  16.05 |  86.72 |   2.25 |  13.38 |  72.27 |   1.69 |  10.03 |   54.2 |   1.13 |   6.69 |  36.13 |

The column headings are "A/S" (Add/subtract) Mul (multiply), amd div (divsision - modulo is similar because it requires division to be performed), followed by and @ and the clock speed in MHz. assuming an AVRxt processor. AVRe+ (pre-2016 revolution) is slightly slower because push is slower. Times are in uS
Speed goes up to 48 MHz because that's how how fast you can overclock a Dx-series part if you use E-spec parts and an external *clock*. I don't know that a crystal would do it, but they should definitely do 40. 
