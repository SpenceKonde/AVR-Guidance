# First, make the hex file human reaable:

## You should use a good text editor (ex, sublime text) capable of regex replaceenmt. 
In these examples the first is what you search for, the second the replacement pattern. 
If there are 16 bytes per line, this will strip off the checksum and line start, which is rarely useful, leaving you with just instructions. 

```
:10[A-F0-9]{4}00([A-F0-9]{8})([A-F0-9]{8})([A-F0-9]{8})([A-F0-9]{8})..
\1  \2  \3  \4 
```
The same for 32-byte lines (output by some tools)
```
:10[A-F0-9]{4}00([A-F0-9]{8})([A-F0-9]{8})([A-F0-9]{8})([A-F0-9]{8})([A-F0-9]{8})([A-F0-9]{8})([A-F0-9]{8})([A-F0-9]{8})..
\1  \2  \3  \4  \5  \6  \7  \8 
```
After that swap endienness to make it "readable":
```
([A-F0-9]{2})([A-F0-9]{2})([A-F0-9]{2})([A-F0-9]{2})
\2\1  \4\3
```
Notice the trailing spaces! They are VERY important!
Add leading ones too:
```
^([^ \n])
 \1

## Here are what some of the big red flags would be

(opcodes all have had the byte order reversed to match how Atmel/Microchip prsent them)

This following applies to anywhere that isn't in a data section (where it represents data rather than code)

## flash size gives a hint
There are a small number of 32-byte isntructioncs which can be a big clue. jmp, call, sts and lds.

The STS and LDS instructions, 0x90x0, 91x0, 92x0 and 93x0 when the following value is an larger than the size of the data space in bytes (see datasheet memories chapert) are load and store instructions to addresses that don't exist on the chip. Unless they are either in a data section, or are directly preceeded by the first word of a 2-word instruction, the program was not valid compiler output, and hence proove you got the bunk file. 

Likewise, jump and call can be a giveaway, these and also 32-bit instructions, with the first word being 0x95ny, where y is C, D, E, or F. A value of n that is not 0 (recall that we are excludin xmegas, the only parts with >256k flash, or a value of y that is D or F on a chip that does not have 256k of flash is not valid compiler output. Om smaller-flash parts, similarly, if the following word is larger than the flash size in words, it is not valid compiler output, because it is attempting to jump or call to a location that does not exist on the part. The same caveat that these values may appear as data or the second word of a 2-word instruction applies. 

There are only 8 opcodes that can be the start of a valid 2-word opcode for parts with 256k of flash or less are 0x9000, 0x9100, 0x9200, 0x9300, as well as 0x940C-0x940F and 0x950C-0x950F - amd the ones ending in D and F are valid omly on 256k parts/ 

9519 and 9419 aren't valid for parts with 128k or less flash (they're EIJMP and EICAL)

If the device is a tinyAVR released prior to 2016, any opcode starting in 0x02, 0x03, and 0x9C, 0x9D 0x9E, or 0x9F are are not valid, because those are multiplication, which these parts don't support. 

0x92xy, and 0x93xy, where y is 4, 5, 6 or 7 are not valid instructiosn on non-xmega processors, nor is 0x94xB. (these are the XCH, LAS, LAC, LAT, amd DES instructions). 

0xFAxy, 0xFBxy, 0xFCxy and 0xFDxy, if y is larger than 8 or higher, are invalid. 

0x95F8 is valid only on AVR Dx Series (SPM with Z postincremented)
0x95E8 and 0x95F8 are never valid in locations where self programming is not enabled:
* Classic AVRs - if self-programing is not enabled, these instructions are doomed to fail. 
* Modern AVRs - If BOOTSIZE/BOOTEND = 0, these are invalid, and if CODEIZE = 0, these are only valid within the bootloader section, and if CODESIZE != 0. they are never valid after the end of the application section. 

## Regex solution (starting from the initial sequence:

Mark all liekly LDS and STS instructions; `9000  1234`  (notice how we get double spaces from above  regexes - load contents of 0x1234 into r0) will be turned into `9000 _1234_`

```
(9[0-3][0-9A-F]0)  ([A-F0-9]{4} ?)
\1__\2_
```

Same for valid jump and call
```
(9[45]0[C-F])  ([A-F0-9]{4} ?)
\1@@\2@
```
Now that we've de-spaced them, they won't match further searches. 

Now scan for invalid jumps and calls:

```
 (9[45][1-9A-F][C-F]) 
X\1X 
```

and XMega-only memory access:
```
 (9[23][0-F][4-7]) 
X\1X
```
Amd DES:
```
 (94[0-9A-F]8)
X\1X
```
And invalid BLD, BST CBI and SBI
```
 (F[89A-F][0-9A-F][89A-F])
X\1X
```

Finally search this modified file for 
```
X[0-9A-F]X
```
Amy matches must be either in data sections, or arean indicationthat the coee is not valid and you got the bunk file. 

## Statistical patters are also useful"
Search
```
F[0-7][0-9A-F]{2}
```
find all, copy, and paste into a new document. If there aren't any of these, it is not a plausible program, as it would mean there were no branch instructions. 
This is all the branch instructions
In new document, replace
```
[0-9A-F]{3}([0-9A-F])
\1
```

Sort in numerical order. 

0, 1, 8 and 9 - branches based on the zero and carry flags, should be very common. 
2, 3, 4, B C and D are uncommon but not unheardof in compiler generated code.
5 and D (half carry tests) rarely if ever generated by the compiler. Frankly even a single one would raise my eyebrows. Large numbers of them, unless the code involved BCD math (and who does that these days?), are a huge red flag! 
6 and E are very rarely used, if ever; the compiler generates the functionally equivalent but more powerful sbrs/sbrc rjmp in almost every case. (in assembly, you're more likely to see it, though it's still rare - it permits the bit to be branched based on to be remembered while other operations may change it before the branch happens (this is similar to how hand written assembly often does things like cp, breq, brcs (compare two values, take one branch if they're equal, otherwise, if carry is set, indicating second value was larger, take a different branchm, and otherwise continue. The compiler rarely if ever will do that, buit it's fairly common when trying to write very tight assembly/
7 and F (based on interrupt bit) is a bizarre thing to branch on, it is almost never seen. 
