# Making AVR math faster

[See the AVR math speed review](AVR_Math_Speed.md)

With those numbers, you can see how in some applications, the speed of math could become limiting.

## Basics

* Use the smallest appropriate datatype.
* Try to do more of the math before things have to be converted to larger datatypes
* Be aware that arduino's random generates 32 random bits, and involves a 4-byte division. Even if you only want a 1 byte value.

## Conquering division
* Don't divide by powers of 2, rightshift instead. Rightshifting a 4-byte value N places takes `4 + 8n`clocks (it sets up a loop to save flash)
* Division can be approximated through addition/subtraction and rightshifting. See wiring.c for some examples in micros, in both C and extensively commented assembly, because the compiler was missing some chances to optimize.
* Sometimes just the way you define a value (as in what units it represents) can help you out by letting you do math around it where the constant you divide it by shakes out as 1 - of course that only helps you if you have a way to make use of this value, like an unavoidable floating point multiplication by a constant that the correction for the convenience unit can be applied.
* You can easily configure the ADC such that 1 ADC LSB = 1 mV. Why divide that by 1000 to get volts? Then you have a float and everything is slower.
* Often you will have something that's in 1024ths, (for example, `RTC.CNT` with a 32.768k xtal counts 32768 ticks per second. Converting it back into human math would typically require division. But there's a trick for integer types.  (val in 1024ths) - (val >> 5) + (val >> 7) = val in 1000ths. The compiler implements this worse than it could. Because we compile with -Os, the compiler obeys you, choosing to save time to save flash over time.
```
uint16_t ticks2millis(uint16_t ticks); //known to never exceed 2^15
// assuming the RTC ISR increments the overflow count and then exits and the chip goes back to sleep unless awoken another way, a uint16_t for overflow count is faster.
// 32.768 kHz clock = 1/32.768 ths
uint8_t oldsreg = SREG;
uint16_t m = rtc_overflow_count;
sreg = oldsreg;
uint16_t ms = 0;
__volatile__ __asm__ (
  "movw %A1, %A0"
  "movw %A2, %A0"
  "lsl %A1"
  "rol %B1"
  "add %A2, %B1" // (16 bit val + (16-bit-value >> 7), (shortcut: 16-bit value << 1 but use only high byte. Since input is constraints to 32767 this is safe.  )
  "lsl %A1"
  "rol %A1"
  "lsl %A1"
  "rol %A1"
  "sub %A1, %B1"
  ""
:"+&r"(m),"+&r"(t),"=&r"(ms)  );
return ms;

```
## You don't want to float
Floats are bad. Floats are bloated and slow. You don't want to be using floats if you can avoid it. They are not quite as slow to divide as a long, but they aren't much better. And unlike other data types, all opperations on floats are slow *and depend on the data* While dividing two longs is typically 1/3rd slower than floats, two long's can be added in about 1/6th the time as two floats, aand multiplication is can be done in half the time with a long than a float (on average).
* Let that part about the data dependance sink in for a minute. The effect is small for adding and subtracting floats, slightly larger for multiplying them, and largest for division. Integer types have dependence on data too for division, but it usually doesn't matter. 64-bit integers have explosive dependance on data. It is not evenly distributed (of the 2^64 possible 64-bit integers, half can be divided very quickly, because the answer is truncated to zero, and it doesn't take very long to notice that, compared to actual division; as soon as the denominator is smaller than the numerator, the speed of division starts falling. A random 64-bit integer divided by a random 16-bit one (which is promoted per c rules) averages around 6 times slower!
* If the floating point value is getting sent to some external device, it's almost certainly not being sent down the wire as a float - the library author decided it should be expressed as floating point (likely because that makes sense considering what the value is, and may not have been aware of the magnitude of the consequences, or they were not serious for his envisioned use case - maybe he had it connected to an ATmega328p or ATmega4809, and you're hoping to use it with an ATtiny with only 4k of flash). Find the point where they convert the float to a format that gets sent down the wire, and replicate that functionality without the intermediary float.

## He who divides will be conquered
Use Ersatz Division, particularly on 32-bit values. 

Appendix 2: Bitshift division
Particularly for 32 bit values, you really do not want to do division when you are doing time critical stuff. If the divisor is constant, you don't have to.
1. Work with unsigned values. 
1. Rightshift until the number is less than the target number. 
2. rightshift until the magnitude of that value minus the previous one will bring the sum a close to the target as possible. Subtract it from original number.
3. Rightshift that intermediary number until the magnitude of that value **plus** the previous one will bring the sum a close to the target as possible. Add them.
4. Repeat 2 and 3 as needed to get required accuracy. 
5. Alternate addition and subtraction to prevent skips and jumps in the values. 
6. All shift values must be constant]
7. Implementing in assembly will dramatically improve performance but use more flash by eliminating the loops that the compiler uses amd allowing you to drop terms once you know you've shifted all the 1's out of them. 
