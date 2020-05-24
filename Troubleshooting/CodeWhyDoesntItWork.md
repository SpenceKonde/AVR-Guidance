# Troubleshooting Your Sketch
As developers, we inevitable spend most of our precious time chasing down bugs in our code. Often, this is simply an exercise in tedium and/or deep pondering to figure out how to fix the bug you understand - but other times, figuring out WHY it's not working can be a challenge. This document describes some techniques that can be helpful for finding and/or understanding bugs

## If the sketch won't compile

### 'variable' was not defined in this scope 
Most often, this is simply an typo in the name of a variable, or you forgot the rules of variable scope in C (google it or use your favorite general C/C++ reference website). However, particularly when working with parts other than the most popular official boards (ie, what these guides are most concerned with), you may encounter cases where the error is occurring in either low-level code or library code and the "variable" is in fact a hardware register that simply doesn't exost on the part you're compiling for, because it either doesn't have the relevant peripheral, or the equivilent register is named something different there. Those follow specific naming conventions, and are supplied by the compiler toolchain (ie, if they're not defined for some part, they ain't supposed to be defined there). If you see these, first double-check that correct board is selected, and, depending on what kind of register it is, consider
#### General pattern
Frequently writes to a register look like `REGNAME|=(1<<RBIT);` or `REGNAME&=~(1<<RBIT);` - Both REGNAME and RBIT0 could be undefined (which is a good sign that the part doesn't have the functionality your code or library relies on) - the first of those sets bit RBIT in REGNAME, and the latter unsets it (if you're confused, look up bitwise operators in your favorite C/C++ resource). RBIT is the number of the bit (0~7) that controls a specific functionality in the relevant peripheral, and RBIT would be the name that the datasheet refers to that bit as, likewise, REGNAME is what the datasheet calls that register.  (eg "If RBIT is set in REGNAME, then such-and-such is enabled and does this").
#### Combination of capital letters, numbers - shorter than 8 characters - ex "TCCR1A"
* Register names for classic AVR devices. 
* If you're working on a tinyAVR or megaAVR 0-series, tinyAVR 1-series, or DA-series, the code or library you're using needs to be rewritten to support the "modern AVR" devices. 
* If you're working on a classic AVR
  * TIFR, TIMSK, TIFRn, TIMSKn, GIFR, GIMSK - these are interrupt flag and mask registers, and the names and layouts of these registers vary a lot between parts. Search for the register in the datasheet for the atmega328p, then go look in the same chapter of the datasheet for the part you're working with and see if you can find the equivilent register (if it doesn't have an equivilent chapter, the device you're working on doesn't have that peripheral). For example, The ATtiny85 puts the interrupt flags from both timer 0 and 1 into a register called TIFR, whereas almost every other classic AVR has a TIFR0 and TIFR1. 
  * `TCCRnA/B/C`, `OCRnA/B/C`, `TCNTn`, `ICRn` - configuration registers for timers - does the part you're working with have timer `n`? Is it the same kind of timer as on the part the code was originally written for?
* In general, you want to search datasheet of the part the code was written for for that register or bit name, see what peripheral it's associated with, and then see if the part you want to use has the same peripheral. 
#### Combination of capital letters and numbers, a `.` or `_`, then another set of capital letters and numbers
* Register names for tinyAVR or megaAVR 0-series, tinyAVR 1-series, or DA-series
* If using a classic AVR, the code is not supported there, it must be rewritten to support those parts; this may not be possible due to the less sophisticated peripherals on the classic AVRs.
* If not using a classic AVR, check the datasheet to see if the peripheral (the part before the underscore) exists on the part you are using. If not, code must be modified to use hardware that your part has - if possible (it may be that the code requires a peripheral that your part simply doesn't have - for example TCD0 on a 0-series part (those are only in 1-series or DA-series).
#### Combination of capital letters, numbers, and underscores, followed by _bm, _bp, _gc, or _gp
* bit position (like the classic bit names) and bit mask (like 1<<BITNAME) for tinyAVR or megaAVR 0-series, tinyAVR 1-series, or DA-series. _gc is the "group code" for a particular setting from a multi-bit field, and _gp is the position of a multi-bit field within a register.
* All above considerations apply.
* If using a "modern AVR", which does have the peripheral - is it referring to a feature that doesn't exist on your part? For example, TCB_CLKSEL_EVENT_gc or TCB_CASCADE_bm on tinyAVR 0-series or 1-series or megaAVR 0-series - those parts don't support clocking TCB's off event or cascading two timers (which depends on clocking the second one on event)


## if the sketch compiles, but doesn't work
