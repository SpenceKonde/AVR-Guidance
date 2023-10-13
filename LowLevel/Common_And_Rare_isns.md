# Which instructions are common and which are rare
Some instructions are very commom. Others are very rarely seen
This will be expanded as more data is examined

## Common instructions
These generally top the list of instructions from assembly listings.
* push and pop, in approximately the same number are very common 
* rjmp
* sbic/sbis (`if (VPORTx.(DIR|OUT|IN|INTFLAGS) & 1 << bitpos)` and similar.
* in/out (reading or writing to the I/O space - less common on modern AVRs, maybe only moderately common there. Still, the SREG is there; `uiny8_t oldsreg = SREG; cli(); (stuff) SREG = oldsreg;` ideom contains an in and an out, as does every interrupt for this reason.) 
* st, std, ld, ldd - writing to and reading from RAM, flash and SFRs. Oddly enough, store is usually more frequent than load. (note that stores to y and z without displacement are not actually a distinct instruction - ld or st on y or z is simply ld y+q or z+q where q == 0.
 
## Moderately common instructions
* sts, lds - Used for register access if not using a pointer (ie, the compiler is not smart enough to see you do `TCA0.SPLIT.CMP0L = 128; TCA0.SPLIT.CMP0H = 128;<repeat for cmp1 and cmp2>` and realize it can use a pointer to implement it more efficiently - this renders as `sts sts sts sts sts sts` (12 words, 12 clocks), rather than ldi [r26/28/30]; ldi [r27,29,31]; st [xyz]+; st [xyz]+; st [xyz]+; st [xyz]+; st [xyz]+; st [xyz]+;` (looks longer, but those are all 1 word instructions, so it's only 8 words, 8 clocks. Though, *if* all pointer registers are full with non-dead data (ie, data which is going to be used again), the pointer implementation gets worse. If there's a known unused register pair it can use, it can just movw the data there and move it back at worst (2 words 2 clocks added, still better), but if there's no register pair either, it would need 2 addoitional pushes and 2 additional pops, bringing it to 12 words and 14 clocks (pop is 2 clocks), and making it strictly worse than the sts implementation. The benefit gets larger the larger the value, but the compiler doesn't seem to pick up on that. Thankfully most of the time when doing intensive register manipulation, where the 
* sbrs, sbrc - Often `if (var & 1<<bitpos)` is rendered as `sbrc var, bitpos; rjmp end_of_conditional` 
* call, rcall - expect fewer of these than places a function is called in c code because of inlining and optimization. 
* ret - expect fewer of these than returns in the c code, because of inlining. 
* eor - almost always with the both arguments being the same lower register (`clr Rd` is `eor Rd, Rd`) (this is how you clear a register you can't ldi 0 into). 
* Arithmatic instructions in general

## Uncommon instructions
You'll usually see a few of these in any largish program. But not many. 
* cbi, sbi, bld, bst - at least outside of inline assembly or code that goes out of it's way to ensure they are used
* icall/ijmp - Most programs have a few of these. Wherever they are found they bring woe and misery to the would-be debugger (generally it's not obvious what the indirect jump and call instructions are pointing to. It takes some registers containing mystery values, throws them into z, and jumps to it. Most often associated with classses and their methods, or places where you explicitly call a pointer. 
* sleep - even in code that makes extensve use of sleep modes, you will likely find only a 1 or a small number of sleep instructions (in fact, if I saw several, I would consider that to be a red flag) and sign that it's sleep logic was likely hiding bugs. 
* swap - the compiler is rarely clever enough to use swap, even when it provides large performance benefits, and where any asm implementation would use it if the author knew what they were doing. It takes some thought to realize why - we write code making implicit assumptions (and hopefully testing them if they may not be valid). If the compiler is asked to render a <<4 of a 1 byte variable, it has to shift it 4 times, because while *you* may know that the value is always going to be less than 16 because it's getting placed into a 4-bit bitfield in a register, and you've already made sure of that, the compiler will frequently miss the possibility, and think that it would need to swap and then clear the unwanted nybble, making it much less favorable. 
* wdr - for the same reason as sleep - even when used, it's generally called in only a couple of places/ 
* reti - only used once per interrupt. 

## Rare instructions
* spm
* bris/bric - who branches on the global interrupt flag?
* brts/brtc - only used in some mathematical operations where bits need to be stored for later, generally routines hand written in assembly as part of avrlibc. 
* bld/bst - almost as above, but not quite. The compiler, in rare cases, is clever enough to spot one of the places where bld/bst are useful (these are not numerous)
* seh/clr/brhs/brhc - when AVR was designed, BCD was a much 


## instructions only seen in hand written assembly code
These are synonyms for other instructions, and the compiler renders the other version
* lsl (add Rd. Rd_
* rol (addc Rd, Rd)
* cbr (andi with bits of constant inverted)
* clr (eor Rd, Rd)
* tst (and Rd, Rd - for that matter, or Rd, Rd would do the same thing)
