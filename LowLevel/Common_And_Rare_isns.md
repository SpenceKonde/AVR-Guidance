# Which instructions are common and which are rare
Some instructions are very commom. Others are very rarely seen
This will be expanded as more data is examined

## Common instructions
These generally top the list of instructions from assembly listings.
* push and pop, in approximately the same number are very common 
* st, std. 
* rjmp
* in/out, sbis/sbic (on classic AVRs and code specfically written to generatethem on modern AVRs)
* st, std
* ld, ldd

## Moderately common instructions
* sts, lds
* sbrs, sbrc
* call, rcall
* ret
* eor - almost always with the both arguments being the same lower register (this is how you clear a register you can't ldi 0 into). 
* Arithmatic instructions in general

## Uncommon instructions
You'll usually see a few of these in any largish program. But not many. 
* cbi, sbi, bld, bst - at least outside of inline assembly or code that goes out of it's way to ensure they are used
* icall/ijmp - Most programs have a few of these. Wherever they are found they bring woe and misery to the wouldbe debugger, thankfully (generally it's not obvious what th indirect jump and call instructions are pointing to. But only a handful.
* Sleep - even in code that makes extensve use of sleep modes, the sleep instruction itself is only used in a one or s handful of places. 
* swap - the compiler is rarely clever enough to use swap, even when it provides large performance benefits.
* wdr - for the same reason as sleep - even when used, it's generally called in only a couple of places/ 
* reti - only used once per interrupt. 
* tst

## Rare instructions
* spm
* bris/bric - who branches on the global interrupt flag?
* brts/brtc - only used in some mathematical operations where bits need to be stored for later, generally routines hand written in assembly as part of avrlibc. 
* bld/bst - as above
* seh/clr/brhs/brhc - when AVR was designed, BCD was a much 


## instructions only seen in hand written assembly code
These are synonyms for other instructions, and the compiler renders the other version
* lsl (add Rd. Rd_
* rol (addc Rd, Rd)
* cbr (andi with bits of constant inverted)
* clr (eor Rd, Rd)
* tst (and Rd, Rd - for that matter, or Rd, Rd would do the same thing)
