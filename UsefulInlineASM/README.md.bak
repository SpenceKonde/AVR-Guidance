# Useful inline ASM
In AVR programming, with the low speeds and limited resources of the parts, one often finds oneself in a position where the standard solutions are just too slow.

## Included in DxCore and megaTinyCore
These macros are already defined for you by DxCore and megaTinyCore.
### Timing based on cycle counting
For longer delays use a loop. Even NOP14 is of dubious utility, but will remain accurate even under extreme register pressure.
```c++
#define _NOP()    __asm__ __volatile__ ("nop")
#define _NOP2()   __asm__ __volatile__ ("rjmp .+0")
#define _NOPNOP() __asm__ __volatile__ ("rjmp .+0")
#define _NOP8()   __asm__ __volatile__ ("rjmp .+2"  "\n\t"   /* 2 clk jump over next instruction */ \
                                        "ret"       "\n\t"   /* 4 clk return "wha? wtf?" */    \
                                        "rcall .-4" "\n\t" ) /* 2 clk rcall "Oh, I see. We jump over a return (2 clocks) rcall it (2 more), and then immediately return, taking 4 more clocks." */
// for 9 clock delay, the 3 clock loop is the smallest.
#define _NOP14()  __asm__ __volatile__ ("rjmp .+2"  "\n\t"   /* same idea as above. */ \
                                        "ret"       "\n\t"   /* Except now it's no longer able to beat the loop if it has a free register. */ \
                                        "rcall .-4" "\n\t"   /* so this is unlikely to be better, ever. */ \
                                        "rcall .-6" "\n\t" )
```
### Math and bit moving
```c++
// swap nybbles of a byte
#define _SWAP(n) __asm__ __volatile__ ("swap %0"  "\n\t" :"+r"((uint8_t)(n)))
// Add or subtract an 8-bit value to/from a 16-bit one, when you *know* that no carry will result
// saves a paltry 1 word and 1 clock.
#define _ADDLOW(a,b) __asm__ __volatile__ ("add %0A, %1"  "\n\t" :"+r"((uint16_t)(a)):"r"((uint8_t) b))
#define _SUBLOW(a,b) __asm__ __volatile__ ("sub %0A, %1"  "\n\t" :"+r"((uint16_t)(a)):"r"((uint8_t) b))
```
## Timing
All of these timing examples assume interrupts are already off, since you need to do that before the start of the timing critical code.
### Standard delay loops
```c++
uint8_t temp;
__asm__ __volatile__ (
  "ldi %0, %1"  "\n\t"
  "dec %0"      "\n\t"
  "brne .-4"    "\n\t"
  :"=d" (temp):"M" (clocks to delay divided by 3)
  );
// pad with rjmp .+0 if delay mod 3 = 2, or nop if delay mod 3 = 1.

// Armored version of above. Under extreme register pressure, or in an ISR, the compiler will have to free up that register.
// This takes that out of the compilers hands by using a fixed register, by saving and restoring it ourselves, since we
// need to know where exactly the delay will occur in timing critical code.
__asm__ __volatile__ (
  "push r24"      "\n\t" // save r24 +1 clock
  "ldi r24, %0"   "\n\t" // 1 clock
  "dec r24"       "\n\t" // 1 clock
  "brne .-4"      "\n\t" // 2 clocks, except last pass, where it takes 1, compensating for the ldi on the first round
  "pop r24"       "\n\t" // 2 clocks to restore r24.
  ::"M" ((clocks to delay divided by 3)-1)
  );
// pad with rjmp .+0 if delay mod 3 = 2, or nop if delay mod 3 = 1.


```

## Mathematics
Note: for the common case where you want to compose a 16-bit value from 2 8-bit ones, just use a union.

### Fast rol32 and ror32
This is a rotate-left opperation on a 32-bit value implemented in 9 words and 9 clocks-per-shift. Note that there is not one bit in limbo with this, as the normal reason to want such an opperation is for xorshift applications which reqire that (typically seen in the context of encryption).

The naive implementation (number << bits)|(number >> (32-bits)) is both larger and always takes the maximum length of time.

You should only use this when 17 > bits > 0. If bits > 16, use ror32 with 32-bits. And if bits == 0, you're done.

```c++
uint32_t rol32(uint32_t number, uint8_t bits) {
  __asm__ __volatile__ (
    "clc"           "\n\t" // make sure carry is cleared when we start.
    "lsl %0A"       "\n\t" // leftshift the low byte
    "rol %0B"       "\n\t" // now second byte
    "rol %0C"       "\n\t" //
    "rol %0D"       "\n\t" // and final byte.
    "brcc .+2"      "\n\t" // we just did an lsl on the lsb, so we know low bit is currently zero, so see if we just shifted out a 1.
    "inc %0A"       "\n\t" // if so add 1 to the lsb. No need to carry because we know the low bit is 0
    "subi %1, 1"    "\n\t" // bits-- and clear the carry flag (unless called with bits = 0 - don't do that, or add a check in the C code.)
    "brne .-16"     "\n\t" // branch back to the start.
 : "+r" (number), "+d" (bits))
 return number;
}

uint32_t ror32(uint32_t number, uint8_t bits) {
  __asm__ __volatile__ (
    "clc"           "\n\t" // make sure carry is cleared when we start.
    "sbrc %0A, 1"   "\n\t" // If we're about to shift out a 1, we set carry
    "sec"           "\n\t" // so we will shift it in the other side.
    "ror %0D"       "\n\t" // rightshift the high byte
    "ror %0C"       "\n\t" // now second byte
    "ror %0B"       "\n\t" //
    "ror %0A"       "\n\t" // and final byte.
    "subi %1, 1"    "\n\t" // bits-- and clear the carry flag (unless called with bits = 0 - don't do that, or add a check in the C code.)
    "brne .-16"     "\n\t" // branch back to the sbrc.
 : "+r" (number), "+d" (bits))
 return number;
}
```
