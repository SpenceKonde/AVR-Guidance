# Example inline assemble
These little snippets of inline assembly can be used so you don't have to write them yourself, as inspiration, or a basis for adaptation. 

All code in public domain. 

## Delays
Everyone knows the trick with "rjmp .+0" to get a 2-cycle noop with a single instruction word. Here are some ways to get longer delays. Note that it doesn't take long before the loops are more flash-efficient - but you may not want to use them. (why might you not want to loop? Good question - all I can think of is corner cases where the cost of using a working register makes life awkward. It would rarely be a straight up performance problem; the issue instead would be that it would be harder to predict before compiling it how the use of a working register for the loop counter would impact performance.... or you would like to avoid a comparison or math operation that would change the SREG. This can also be handy when you just don't want the complication of picking up after a little loop of assembly in the middle of the already-annoying-to-write inline assembly routine you're working on)

Of course, before you go using something like this, think about whether you woudldn't be better off using delay.h's `_delay_us()` which will correct for F_CPU - though it does require that the delay time be a compile time known constant.

```c
  __asm__ __volatile__ (
    "rjmp .+2" "\n\t"     // 2 cycles - jump over next instruction.
    "ret" "\n\t"          // 4 cycles - rjmped over initially...
    "rcall .-4");         // 2 cycles - ... but then called here
                          // wait 8 cycles with 3 words

  __asm__ __volatile__ (
    "rjmp .+2" "\n\t"     // 2 cycles - jump over next instruction.
    "ret" "\n\t"          // 4 cycles - rjmped over initially...
    "rcall .-4" "\n\t"    // 2 cycles - ... but then called here...
    "rcall .-6");         // 2+4 cycles - ... and again here
                          // wait 14 cycles with 4 words                
    // More generally, wait 8 + 6*N cycles in 3+N words - 14 in 4, 20 in 5, etc. 
    uint8_t cycles_divided_by_3 = 10; // 1 cycle, assuming you have a working register...
  __asm__ __volatile__ (
    "1: dec %0" "\n\t"  // 1 cycle
    "brne 1b" : "=w" (cycles_divided_by_3) : "0" (cycles_divided_by_3)); // 2 cycles, 1 if false.
  // Generally 3 words - except if you let the compiler allocate the register, you of course don't know that it wont use more
  // due to having to push something onto the stack and then pop it back after. 
  // If it's in the middle of an assembly routine and you know there's a register that you don't care about anymore, then you
  // can just use that.
  // That is the classic 3 cycle AVR loop for 3 to 768 clock cycles (counting the ldi at the beginning; start it at 0 to get the maximum).

  // And the 4 cycle one that can run up to 2^16 times:
  uint16_t cycles_divided_by_4 = 1000; // 2 cycles if loading a constant. 
  __asm__ __volatile__ (
    "1: sbiw %0,1" "\n\t" // 2 cycles
    "brne 1b" : "=w" (cycles_divided_by_4) : "0" (cycles_divided_by_4) // 2 cycles
  );
  // gives a delay of 6 to 4*2^16 + 2 clock cycles in just 4 clock cycles. 
  // Either of these delay loops can be extended by adding noop functions (nop, rjmp .+0, etc) to the loop, too. 
```
