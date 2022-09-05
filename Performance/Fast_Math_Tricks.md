# Making AVR math faster

[See the AVR math speed review](AVR_Math_Speed.md)

With those numbers, you can see how in some applications, the speed of math could become limiting.

## Basics
* Use the smallest appropriate datatype.
* Try to do more of the math before things have to be converted to larger datatypes
*
## Division is bad
* Don't divide by powers of 2, rightshift instead. Rightshifting a 4-byte value N places takes `4 + 8n`clocks (it sets up a loop to save flash)
* Division can be approximated through addition/subtraction and rightshifting. See wiring.c for some examples in micros, in both C and extensively commented assembly, because the compiler was missing some chances to optimize.
* Sometimes just the way you define a value (as in what units it represents) can help you out by letting you do math around it where the constant you divide it by shakes out as 1 - of course that only helps you if you have a way to make use of this value, like an unavoidable floating point multiplication by a constant that the correction for the convenience unit can be applied.
* You can easily configure the ADC such that 1 ADC LSB = 1 mV. Why divide that by 1000 to get volts? Then you have a float and everything is slower.
* Often you will have something that's in 1024ths, (for example, `RTC.CNT` with a 32.768k xtal counts 32768 ticks per second. Converting it back into human math would typically require division. But there's a trick for integer types.  (val in 1024ths) - (val >> 5) + (val >> 7) = val in 1000ths. The compiler implements this worse than it could. Because we compile with -Os, the compiler obeys you, choosing to save flash over time, implementing the shifts as loops

The generic algorithm is presented further down.

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
Don't let that happen to you. Whenever speed counts, division is not an option.

Appendix 2: Bitshift division
Particularly for 32 bit values, you really do not want to do division when you are doing time critical stuff. If the divisor is constant, you don't have to.
1. Work with unsigned values.
1. Rightshift until the number is less than the target number.
2. rightshift until the magnitude of that value minus the previous one will bring the sum a close to the target as possible. Subtract it from original number.
3. Rightshift that intermediary number until the magnitude of that value **plus** the previous one will bring the sum a close to the target as possible. Add them.
4. Repeat 2 and 3 as needed to get required accuracy.
5. Alternate addition and subtraction to prevent skips and jumps in the values.
  a. You can replace any term with two terms, both with a shift distance differing by 1, where the replaced term is one of those with the opposite sign. This allows you to drive down error caused by unbalanced terms at a cost of having to use more of them.
6. All shift values must be constant.
7. The compiler with -Os will produce code that is both smaller and slower. The larger the shifts, the more pronounced this effect. Assembly can improve this significantly.
  a. There are two ways to implement a series of shift operations (we will use a uint16_t being rightshifted 3 places, and stored in the (unrealistic) r0 and r1 pair.
    ```
    // uint16_t x >>= 3;
    lsr r1
    ror r0
    lsr r1
    ror r0
    lsr r1
    ror r0
    ```
    Pretty simple. Flash = four times the size of the datatype per shift in bytes. Runtime = Twice the size of the datatype per shift in clocks.
  b. That's not wht the compiler gives you, it gives you:
    ```
    ldi r18, 3 //or any other available upper register
    lsr r1
    ror r0
    dec r18
    brneq .-6
    ```
    Flash = 6 + size of datatype for any number of shifts, runtime = (3 + size of datatype) per shift. So we go from 6 clocks to 15, and flash goes from 12 bytes to 10. This is probably fine, desirable even, when you aren't in a time critical situation. But sometimes you are, and you want to strangle the compiler over this one. If you're a core author with any speeds that are not powers of 2 MHz, you will slam into this when you have to make millis work with the non-power-of-2 speeds. You can't use division (you wouldn't belive the whining when your micros() implementation takes 50-100us to return), and before you implemented the division, you had an angry guy complaining that micros was totally busted, because it would count up to 700-something and then leap ahead to 0 when millis incremented (this is what the official core does if you try to just run it at speeds not a power of 2). Ersatz division is slow, but not nearly as slow as division, and hand implemented ersatz division is better than compiler generated ersatz division.

    | shift (bits) | uint16_t flash | uint16_t time | uint32_t flash | uint32_t time |
    |--------------|----------------|---------------|----------------|---------------|
    |      a >> 1  |        4       |      2        |       8        |      4        |
    |      a >> 2  |        8       |      4        |       16       |      8        |
    |      a >> 3  |    12 vs 10    |   6 vs 15     |    12 vs 10    |  24 vs 21     |
    |      a >> 4  |    16 vs 10    |   8 vs 20     |    32 vs 14    |  16 vs 28     |
    |      ......  |    ........    |   .......     |    .........   |  ........     |
    |     a >> 13  |                |               |                |               |
    |     a >> 14  |                |               |                |               |
    |     a >> 15  |                |               |                |               |
    |     a >> 16  |                |               |                |               |

shifting N-4 bits or more on a datatype of N bytes suddenly drastically reduces flash use and time because the compiler can throw away almost everything and maybe use swap and andi and make much more efficient code. Obviously multiples of 8 bits shift VERY quickly because you just move bytes around.

The compiler is NOT smart when you have series like ersatz division eg `m - m >> 2 + m >> 3 - m >> 6` and so on - specifically, it doesn't seem to be able to recongize that it's easier to get m >> 3 from m >> 2 and so discards m >> 2 then recalculates m >> 3, and then does the same thing for m >> 6. This cannot be worked around short of inline assembly: no matter what steps you take to make it look different, the compiler is clever enough to realize that they're equivalent, and at the point when it is choosing the best implementation, I would wager that it doesn't know it's on a machine that can only shift bits one at a time (most architectures can single-instruction shift an arbitrary number of bits). Hence despite being clever enough to find any possible rephrasing of that math and turn it into the same thing, it's not smart enough to recycle the intermediate values. Thats why wiring.c has two big blocks of inline asm in it on my modern AVR cores - I reused the intermediaries. And then found another trick at the end:

Always, there are patterns that you know about, but that the compiler doesn't and can't, and there are assumptions it can say nothing on but you know to be true. Say you know that the input such an ersatz division routine is a number not greater than 5000. The compiler doesn't know that and can't assume it, but you know that if it is higher than that, it doesn't matter what number you get out of the math because you're putting garbage values in and will get garbage out (GIGO). Therefore, once you're rightshifted it 5 places, the intermediary can't possibly exceed 255 and the MSB is 0. So (when doing it assembly) you can stop using the MSB in your math, because you know it's zero.
