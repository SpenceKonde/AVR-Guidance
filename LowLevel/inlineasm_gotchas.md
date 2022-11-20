# Common inline ASM mistakes

## Wrong constraints which *usually work*
When you screw up the constraints badly enough - like if you use "I" instead of "r" because you confused the opperand with the value you're planning to LDI in the routine with register you're putting in, you find out the next time you compile it. There's the usual annoying problem of findng where in your code it's talking about. But usually, if you're test compiling frequrntly, you won't need such measures. 

But there are cases where incorrect constraints will work most of the time - but then suddenly start failing to compile, or worse, causing strange and subtle misbehavior at runtime when something unrelated is changed inthe code. "What? I've been using this block of assembly for years, why does it suddenly not work?"

### Brief background
AVR has "call clobbered" and "call saved" registers. When C code calls another function written in C, and that function is not inlined there is a list of working registers that it can do what it wants with ("call clobbered") and those wheich it must save and restere. (yes, I know you're wondering how this relates to above.) 

 * r0 is `__temp_reg__` and may be clobbered freely. Always use this for your first temporary register in a block inline ASM. 
 * r1 id `__zero_reg__` which always contains 0 while running C code. Any inline assembly that clobbers it must make sure to set it back to 0 before returning to the C code. You can't `ldi` it because it's in a lower register, but you don't want to set it to any number, you want to set it to 0. Just xor it with itself: `eor r1, r1`
* r2-r17, and r28+r29 (Y pointer) are "call saved" - normally you don't have to think too much about this, because if you ask for a register with a constraint, and it gives you one of these because it meets the constraint, it will do that behind your back. Putting these in your clobber list will add instructions to save and restore them.
* r18-r27 and r30-r31 are "call used". Calling functions must assume that any function call will bend, fold, spindle and mutilate these. the only reason your're not allowed to just pick these up and use them within inline assembly bnlocks is that within the current function, they may have been put to some use, and you could make a mess of that. They do not generate extra overhead if they're in the clobber list unless they were actually being used by the current function.  
* r28-r29 generally contains a pointer to a stack frame. Not that this is relevant now. 
* Arguments to functions get passed in registers r24 downwards as far as r8 (if you have more arguments than that, everything is passed on the stack. 

### How this is related
The compiler tends to assign higher numbered registers first within functions. 

There are (at least) three examples where a wrong constraint will potentially work anyway:
1. If the instruction requires an upper register because you're going to use it with ldi, cpi, or other "immediate" instruction. now imagine your inline ASM constitutes the majority of a function. It may work every time! If it enters the asm block and gets to your problematic constraint while it hasn't used up all the upper registers, it'll assign one and the assembler later on will be none the wiser. 
2. The same thing can happen with sbiw and adiw. (subtract and add immediate from/to word) It should have the =W or more likely +W constraint. But as long as it's allocatted early in a function, you'll get away with +d or sometimes even +r (I've never seen it stated anywhere, but I also have never seen it place a 2 byte operand's low byte in an odd-numbered byte, and I don't think this ever happens because of tings like movw, as well as adiw and sbiw which act on rN and rN+1 - but rN must be even).
3. Finally, at the most extreme end, you can even end up with things that need an 'e' constraint - pointer registers only - getting one by luck when registers are doled out. 

Those may work in nearly every case - until you end up with the compiler inlining one, and now it's in the middle of a much larger function which has already used the choice registers. Now your incorrect constraint leaves the compiler unaware of the true needs of your code, and it now it fails to meet them, and you wonder why your "known working code" suddenly stopped compiling. I will note that since most functions don't use all the call-used registers. Sometimes eliminating the handling of these types of registers can be the biggest beneit of inlining! But, it does have the potential to break "wrong" ASM that worked in other contexts. The optimizer may inline almost any function, not just "inline" ones (try calling digitalWrite() within a sketch starting with 0 "digitalWrites" and adding them one at a time, noting the increase in flash. Start with a function that does `pinMode(GPIOR0,INPUT)` - That will force it to include the pin and port lookup tables, since GPIOR0 is 'volatile' and the compile has to assume it could take any value, and changewithout code the compiler is aware of acting on it, and make the digitalWrite comparison more informative. You'll notice the first one or two digitalWrite calls are big, then it hits a point where each extra ccall only adds 2 or 4 bytes - at that point it's no longer inlining it. 

However, the optimizer's kyyptonite is C++ classes; it seems to result in almost nothing getting inlined or otherwise optimized!. That's why adafruit neopixel, and tinyNeopixel have worked for YEARS with nobody noticing that show() ldi's into a register with constraint +r. (it also calls an operand that it doesn't write a read/write opperand and a register that it does change read-only! Described below....) Both of these things it has gotten away with only because of what the C++ does to the optmizer, since show is a method). 

### In Summary
Make sure your constraints are as constraining as they are supposed to be! +W or =w for sbiw/adiw, r or +r for anything else with "immediate" in the name of the instruction, and e for anything that needs a pointer reg. 

## Don't write a read-only operand 
**This includes anythign that changes it's value!** This will sometimes work (if the variable is going out of scope after the the inline asm block), and other times cause bizzare and confusing bugs. 
This is extremely important for pointer registers:

```asm
__asm__ __volatile__ (
"ld r0, X+,      \n\t"
"ld r1,  X+      \n\t"
"st Z, r0       \n\t"
"std Z+1, r1    \n\y"
"eor r1, r1     \n\t"
: "+z" ((uint16_t) &dest): "x" ((uint16_t) &source) );
```

Both of those constraints are WRONG! 

The z register can be read only. You aren't changing the Z register's value. The thing it's *pointing at* must be declared volatile, because that is being changed where the compiler isn't aware of it. On the other hand, the X pointer which you're only reading from... you're using postincrement on it! That's called changing it! The data pointed at by the X register isn't any different after that inline assembly, but X has increased by 2 and isn't pointing at the thing the compiler belives it to be pointing to. 

## Calling functions from asm is not a good idea
Why? 
Well, a function is allowed to trash any of the call used registers. You don't know which ones, unless the function you are calling is itself contains nothing but a chunk of inline asm, and none of the operands are anything other than the arguments you passed to the function, and the registers they get put in (based on the calling conventions as detailed in the [avr-gcc abi documentation](https://gcc.gnu.org/wiki/avr-gcc) (a very terse document, quick to read, longer to comprehend)) meet the constraints you specify in that inline asm, which you need to know to pass arguments to it anyway. And then you need to recognize and take countermeasures in your routine to make sure that any registers it clobbers, you declare clobbered. 

Or you save and restore every call used register, which probably defeats the purpose of using inline asm, since you're doing it most likely to either save flash, or execute quickly. If you're calling a C function, this is your only option, because you don't know what the compiler will assign to the local variables in the C function. 28 instruction words and 42 clocks of execution time is not helpful for saving flash or speeding execution. So like, you should really try to uh, not call functions from your inline asm. 
