# Reading hex files
Reading hex files by hand is terrible. Try to do everything you can to avoid it!

If you really do have a hex file that contains valuable information for some reason, and you need to figure out something about it... or maybe modify it:
1. Make a backup
2. Load up something that can do regexs substitutions (I use sublime text and love it)

Maybe it's some compiled piece of proprietary - yet free - software that does something incredibly useful like acting as a native USB UPDI programmer, and it runs on a processor that's common and cheap to buy as an assembled board, like a 32u4, and it would be just perfect, except that there's a tiny hardware difference between the common boards, and the one it was designed for. Maybe it, oh, uses the analog voltage reference, which floats on the pro micro but is tied to Vtarget on the hardware this code was written for...

Well - without actually learning any fancy tools, one (*can* manually ("mentally execute") a hex file. It's unpleasant, and slow, and tedious, but sometimes, you've gotta do what you've gotta do. Obviously, you want to narrow down what you have to do as much as possible. Is there a register in the IO space (lowest 64 addresses) that gets read or written which you care about? That'd be an `in` or `out`... Is the pivotal moment a bitflip in the low IO space (lowest 32 addresses)? That could be a `cbi` or `sbi`... Either way, you can calculate what the opcode would be. Registers above 0x0040 (64) are written to (generally) through `sts` and read with `lds` (though `std` and `ldd` are possibilities, as is plain old `st` and `ld` - but usually the compiler isn't smart enough to use the the displaced load/stores, even when we're poking at a peripheral enough that it would be favorable... It's much more willing to use it to access SRAM it seems. Which, for this purpose is good, because you don't want to try to find the place in the hex file where a displaced store is used to write a hardware register!

But first, we need to make it readable...

## Using RegEx Find and Replace:
In the following find/replace snippets, you may want to 'view raw' - the spaces around the text are critical to making them work cleanly, every time.... (that's one of the handy things about them, how well they work with the same character that our eyes find to be the best separator.

Find:
```
(:[A-F0-9]{2})([A-F0-9]{4})00(.*)[A-F0-9]{2}
```
Sub with: (NO SPACE at start of line!)
```
\2  \3
```
This gets rid of the stuff that gets in the way of reading it:
 * The checksums at ends of lines.
 * The byte-count at starts of lines.
 * The 00 for normal data and puts a space between address and data.


Find (Yes space at the start): - this keeps it from grabbing the part before the address from last sub, without anything to distract your eye ;-)
```
 ([A-F0-9]{4})([A-F0-9]{4})?([A-F0-9]{4})?([A-F0-9]{4})?([A-F0-9]{4})?([A-F0-9]{4})?([A-F0-9]{4})?([A-F0-9]{4})?
```
Sub with (starts and ends with a space; very important:
```
 \1 \2 \3 \4 \5 \6 \7 \8
```
Repeat above if needed to get the other half of the columns separated into words. Now the key step that makes all the difference....
Find (space at start - so it won't match addresses):
```
 ([A-F0-9]{2})([A-F0-9]{2})
```
Sub with:
```
\2\1
```
This swaps the high and low bytes so that they're in the order that matches the AVR instruction set manual.

## So... Okay..... great... now what....

The table below shows the hex values of all instructions
Where a digit is always that way for a given instruction, it's filled in on left

Annotated/most useful

```
Looking for where it interacts with a certain register because you need to change that
but don't have the source? This is almost always an lds or sts instruction, and will
preceed the address of the register.

90_0 1001 000r rrrr 0000 lds (followed by address)
91_0 1001 000r rrrr 0000 lds (followed by address)
92_0 1001 001r rrrr 0000 sts (followed by address)
93_0 1001 001r rrrr 0000 sts (followed by address)

9409 1001 0101 0000 1001 ijmp
9509 1001 0101 0000 1001 icall
940E 1001 0100 0000 1110 call (<128k flash, followed by address)
940C 1001 0100 0000 1100 jmp (<128k flash, followed by address)
On 256k devices (atmega2560), the last digit can be F or D as well.

B    1011 0PPd dddd PPPP in
B    1011 1PPr rrrr PPPP out
C    1100 LLLL LLLL LLLL rjmp
D    1101 LLLL LLLL LLLL rcall

The conditionals all start with F or 99/9B (skips), and not F8-FB, those are bitsets. FC-FF are skips, too
99   1001 1001 pppp psss sbic
9B   1001 1011 pppp psss sbis

F    1111 00ll llll lsss brbs (also all other br__
F    1111 01ll llll lsss brbc

F    1111 100d dddd 0sss bld
F    1111 101d dddd 0sss bst
F    1111 110r rrrr 0sss sbrc
F    1111 111r rrrr 0sss sbrs

Singletons that are either important, common, or both.
9508  1001 0101 0000 1000 ret
9518  1001 0101 0001 1000 reti
9588  1001 0101 1000 1000 sleep
95A8  1001 0101 1010 1000 wdr
95E8  1001 0101 1110 1000 spm

The rest of the 1-first-letter-per-instruction
E    1110 KKKK dddd KKKK ldi

(not super useful - except that you can quickly determine what the opcode does from just first digit)
3    0011 KKKK dddd KKKK cpi
4    0100 KKKK dddd KKKK sbci
5    0101 KKKK dddd KKKK subi
6    0110 KKKK dddd KKKK ori
7    0111 KKKK dddd KKKK andi

8/A is an ldd or std

Only call/rcall/icall, ret/reti, jmp/rjmp/ijmp, and the Branch instructions can do anything other than increment the program counter
```

Numerical order
```

0000 0000 0000 0000 0000 nop
00   0000 0000 xxxx yyyy unused - 255 unused isn here
01   0000 0001 dddd rrrr movw
02   0000 0010 dddd rrrr muls
03   0000 0011 yddd zrrr mulsu/fmul*
04~7 0000 01rd dddd rrrr cpc
08~B 0000 10rd dddd rrrr sbc
0C~F 0000 11rd dddd rrrr add lsl (when r=d, lsl)

10~3 0001 00rd dddd rrrr cpse
14~7 0001 01rd dddd rrrr cp
18~B 0001 10rd dddd rrrr sub
1C~F 0001 11rd dddd rrrr adc (when r=d rol)

20~3 0010 00rd dddd rrrr and
24~7 0010 01rd dddd rrrr eor (when r=d, clr)
28~B 0010 10rd dddd rrrr or
2C~F 0010 11rd dddd rrrr mov


3    0011 KKKK dddd KKKK cpi
4    0100 KKKK dddd KKKK sbci
5    0101 KKKK dddd KKKK subi
6    0110 KKKK dddd KKKK ori
7    0111 KKKK dddd KKKK andi

8    10q0 qq0r rrrr bqqq ldd (also A0~A7) - though note that when q = 0, this is ld itself, b chooses between Y and Z registers.
8    10q0 qq1r rrrr bqqq std (also A8~AF)


90, 91 ld/lds/lpm, pop
90~1 1001 000r rrrr 0000 lds (followed by address)
90~1 1001 000r rrrr 0001 ld Z+
90~1 1001 000r rrrr 0010 ld -Z
90~1 1001 000r rrrr 0011 - unused
90~1 1001 000r rrrr 0100 - LPM Z
90~1 1001 000r rrrr 0101 - LPM Z+
90~1 1001 000r rrrr 0110 - ELPM Z
90-1 1001 000r rrrr 0111 - ELPM Z+
90-1 1001 000r rrrr 1000 - Unused
90~1 1001 000r rrrr 1001 ld Y+
90~1 1001 000r rrrr 1010 ld -Y
90~1 1001 000r rrrr 1011 - Unused
90~1 1001 000r rrrr 1100 ld X
90~1 1001 000r rrrr 1101 ld X+
90~1 1001 000r rrrr 1110 ld -X
90~1 1001 000r rrrr 1111 pop

92, 93 store equivalents
92~3 1001 001r rrrr 0000 sts (followed by address)
92~3 1001 001r rrrr 0001 st Z+
92~3 1001 001r rrrr 0010 st -Z
92~3 1001 001r rrrr 0011 - unused
92~3 1001 001r rrrr 0100 - unused except on AVRxm for XCH
92~3 1001 001r rrrr 0101 - unused except on AVRxm for LAS
92~3 1001 001r rrrr 0110 - Unused except on AVRxm for LAC
92~3 1001 001r rrrr 0111 - Unused except on AVRxm for LAT
92~3 1001 001r rrrr 1000 - Unused
92~3 1001 001r rrrr 1001 st Y+
92~3 1001 001r rrrr 1010 st -Y
92~3 1001 001r rrrr 1011 - Unused
92~3 1001 001r rrrr 1100 st X
92~3 1001 001r rrrr 1101 st X+
92~3 1001 001r rrrr 1110 st -X
92~3 1001 001r rrrr 1111 pop
320 unused opcodes, or 192 if you don't count the xmega combined load-stores that only xmega gets

In this block, the LSN plays the starring role in choosing the instruction, while the MSB is constant except for it's lowest bit. In many cases this is the high bit of the number of the working register it's opperating on, and the upper MSN of the low byte is the rest of the register number.
94-95_0   1001 010d dddd 0000 com
94-95_1   1001 010d dddd 0001 neg
94-95_2   1001 010d dddd 0010 swap
94-95_3   1001 010d dddd 0011 inc
94-95_3   1001 010d dddd 0100 unused?
94-95_5   1001 010d dddd 0101 asr
94-95_6   1001 010d dddd 0110 lsr
94-95_7   1001 010d dddd 0111 ror
94_8      1001 0100 xxxx 1000 se_/cl_ for operating on the SREG
95_8      1001 0101 xxxx 1000 Assorted argumenttless isns,
     9508 1001 0101 0000 1000 ret
     9518 1001 0101 0001 1000 reti (followed by 6 apparently unused opcodes)
     9588 1001 0101 1000 1000 sleep
     9598 1001 0101 1001 1000 break
     95A8 1001 0101 1010 1000 wdr
     95B8 1001 0101 1011 1000 unused?
     95C8 1001 0101 1100 1000 LPM (implicit r0, may not be supported on recent parts.)
     95D8 1001 0101 1101 1000 ELPM (implicit r0, may not be supported on recent parts.)
     95E8 1001 0101 1110 1000 spm
     95F8 1001 0101 1111 1000 spm Z+
9409      1001 0100 0000 1001 ijmp
9419      1001 0100 0001 1001 eijmp    - followed by 14 unused opcodes
9509      1001 0101 0000 1001 icall
9519      1001 0101 1000 1001 eicall   - followed by 14 unused opcodes
94-95_A   1001 010d dddd 1010 dec
94_B      1001 0100 KKKK 1011 unused except on xmega for DES -16 conditionally unused
95_B      1001 0101 xxxx 1011 unused    - 16 unused
940C-94FD 1001 010k kkkk 110k jmp (followed by address) only 940C used on 128k or smaller parts. 2560k parts can use 940D, and 512k xmegas 941C and 941D too.
940E-94FF 1001 010k kkkk 110k jmp (followed by address) only 940E used on 128k or smaller parts. 2560k parts can use 940F, and 512k xmegas 941E and 941F too.
950C-95FD 1001 010k kkkk 110k jmp (followed by address) None of these are used, would only be needed for an 8 mbyte flash chip, which I doubt is going to happen.
950E-95FF 1001 010k kkkk 110k jmp (followed by address)  none of these ever used
     95B8 1001 0101 1011 1000 unused?
     95C8 1001 0101 1100 1000 LPM (implicit r0 destination)
     95D8 1001 0101 1101 1000 ELPM
     95E8 1001 0101 1110 1000 spm
     95F8 1001 0101 1111 1000 spm Z+
9409      1001 0100 0000 1001 ijmp
9419      1001 0100 2000 1001 eijmp    - followed by 14 unused opcodes
9509      1001 0101 0000 1001 icall
9519      1001 0101 1000 1001 eicall   - followed by 14 unused opcodes
94-95_A   1001 010d dddd 1010 dec
94_B      1001 0100 KKKK 1011 unused except on xmega for DES
95_B      1001 0101 xxxx 1011 unused    - 16 unused
940C-941D 1001 010k kkkk 110k jmp  (followed by address) only 940C used on 128k or smaller parts. 256k parts can use 940D, and 512k xmegas 941C and 941D too.
940E-941F 1001 010k kkkk 111k call (followed by address) only 940E used on 128k or smaller parts. 256k parts can use 940F, and 512k xmegas 941E and 941F too.
95_C-95_D 1001 010k kkkk 110k jmp  (followed by address) Invalid jump to address far too high.
95_E-95_F 1001 010k kkkk 111k call (followed by address) Invalid call to address far too high.


512 instuctions in 94 and 95,of which 32+7+14+14+16 are totally unused - leaving 83 unused, plus 16 more (DES) or xmega only.
120 out of 128 opcodes for jump/call have not been used on any production part.
So at most there are 219 opcodes here.
At worst, there are but 83.


96        1001 0110 KKdd KKKK adiw
97   1    1001 0111 KKdd KKKK sbiw
512 opcodes for the word operations

98        1001 1000 pppp psss cbi
99        1001 1001 pppp psss sbic
9A        1001 1010 pppp psss sbi
9B        1001 1011 pppp psss sbis
1024 opcodes for all the (s?)bi[sc] stuff for high performance operations on the low I/O space.

9C-9F     1001 11rd dddd rrrr mul

A0~7      10q0 qq0r rrrr bqqq ldd
A8~F      10q0 qq1r rrrr bqqq std

B0~7      1011 0PPd dddd PPPP in
B8~F      1011 1PPr rrrr PPPP out

C         1100 LLLL LLLL LLLL rjmp
D         1101 LLLL LLLL LLLL rcall
E         1110 KKKK dddd KKKK ldi

F0-F3     1111 00ll llll lsss brbs
F4-F7     1111 01ll llll lsss brbc
generic for branch instuctions better known by more intuitive names.

F8-F9     1111 100d dddd 0sss bld
F8-F9     1111 100d dddd 1sss - unused, if below is true, likely acts like bld
FA-FB     1111 101d dddd 0sss bst
FA-FB     1111 101d dddd 1sss - unused, if below is true, likely acts like bst
FC-FD     1111 110r rrrr 0sss sbrc
FC-FD     1111 110r rrrr 1sss - unused, if below is true, likely acts like sbrc.
FE-FF     1111 111r rrrr 0sss sbrs
FE-FF     1111 111r rrrr 1sss - unused, said to asts like sbrs
And these are bit-level access to registers

1024 unused opcodes where bit 3 is 1, the largest cluster

Apparently 0xFF is treated an sbrs though - skip if bit 7 in register 31 is 1. That strongly suggests that bit3 gets ignored in that last block (which would be the natural result of there not being a reason to check it because there wasnt;t an alternative action, which in turn implies the rest of this block behaves the same way.

```

## Okay, I found what needs to change
Some time I'll write about the process of hackign something up in greater detail; key points are:

Try to just change a value. If you can't do that, try to just change similar instructions.
If you can'd do either of those things, can you tinker with existing control flow  to get what you want? There is an unfortunate possibility that you might need to actually ADD code. Which is particularly painful, but do-able: replace two instructions with a `call` to empty flash, and proceed to act **almost** like you're writing a naked ISR. Because you're writing raw application code, there is no ABI or concept of call used registers. Any register you cannot determine is unneeded at the time of making that call must be preserved. Same goes for the SREG! Be sure to include those two instructions that you had to displace to make room for the call. The only thing different from writing an ISR is if there's a value in one of the registers and that's what you need to modify, obviously you don't want to save or restore that one!

The importance of minding the SREG is hard to overstate: all control flow is based on it, with the exception of two families of "skip-ifs" - hoywever, you can look ahead from the instructions you altered, one at a time. If you reach a br__, adc, sbc, sbci, rol, ror, etc (something that depends on the SREG) before you reach an instruction that blows away the SREG bits other than T and I, then yes you must absolutely preserve and restore it. If you reach one of those SREG clobbering instructions first, then you don't need to (note, however, that inc and dec are NOT sreg clobbering - they only clobber parts of it. And you already know if your code is going to be using the T or I bits.

## Yes, this is viable!

Do you think this whole thing is silly and impractical? That it's not good for anything except playing armchair-instruction-set-engineer?
Remember the opening story of the 32u4 programmer code that's available as hex but not code? That's Microchip's mEDBG. And the bit about `lds` and `sts`being particularly useful?
That's how https://github.com/MCUdude/microUPDIcore/commit/49878acda00474831b03005a3cebbdd498bb4b8c#diff-299fd66b54cb0c05b308b3969047fdd0e17f132cedfd167008fb192dc303a19e happened, and (what immediately prompted it was that, although I consider myself pretty good at soldering - I found the hardware mod to make the stock f/w work to be nearly impossible to achieve. It went down something like this:

"Fuck this soldering, I'm good at soldering, and this is harder than soldering a padless QFN with an iron. I've been at this for a half hour, got it connected twice, and both times the connection broke before I could glue it in place. Why the hell are we using inappropriate code ANYWAY?! Let's look at the 32u4 datasheet, what can enable external AREF? Hmm, there's only one register on the whole chip that controls whether AREF is used... now that we reformatted the hex so we can read it, let's search for the address - oh, here, 1 match! And sure enough, that's an STS immediately before it - so this must be where it gets configured to use the external reference! And it says it's storing r16 to it... uhoh, what value is in r16? could be anything, and how could we clear the key bit without breaking something else? That's not gonna be fun... but what's the instruction right before that one? Starts with E, that's LDI, second digit 0 - well will ya lookit that, it's an ldi into r16! And indeed, that value being loaded to that register would enable external VREF. Then, I opened up a clean copy of the file, searching for the swapped-order bytes I needed to change, turned the appropriate 1 into a 0, adjusted the checksum, and Bam! Problem solved!" Took less time than I'd already wasted trying to do the hardware mod!

## Appendix A - for armchair ISA designers
ISA = Instruction Set Artchitecture

If - like any engineer-type - your first thought is "Huh, those guys made such a bad decision, I could totally do it better" - maybe you even day-dream about how you'd design a better architecture... Well, it's way better to have some familiarity with an example of what you imagine designing, right?

One thing that's interesting to note is the distribution/usage of opcodes (or rather, which instructions eat up huge chunks of the 16-bit number for each instruction)
* 3/8ths of the instruction-space is taken up with `xxxx-immediate` instructions! (ever wonder why only work on the top 16 registers? Now you know - not enough bits in a 16-bit instruction to let these take up any more of the instruction-space than they already do
* 1/8th load/store with displacement (well, if you were ever wondering why they don't have an X-register version of those, there's your answer!
* rjmp and rcall are each 1/16th of the instruction-space, and in/out is another 16th.
* At this point we are up to 11/16ths.
* 3/16ths or so math and logical/arithmatic operations (actually, 13/64ths. bringing us to 14.25/16ths full
* 1/8th is load/set with displacement (well, if you were ever wondering why they don't have an X-register version of those, there's your answer!
* rjmp and rcall are each 1/16th of the instruction-space, and in/out is another 16th.
* 3/16ths or so math and logical/arithmatic operations.
* 1/32nd conditional branch, 1/64th skip-if
* That means 15/16ths of possible opcodes are used, and we haven't even done any instructions other than the conditional branches, immediates, skipifs, logical/arithmatic, load/store with displacement (but not load/store without displacement), in/out and rjmp/rcall. A few remaining instructions are 256 or 512 opcodes. (1/256 or 1/128th of possible opcodes.)

### Slack space calculations
So how much slack is left in the instruction set? Very, VERY little!
```text
By the first letter of the opcode:
0 - 0x0001~0x00FF are not used, 255 instructions.
1 - Full
2 - Full
3 - Full
4 - Full
5 - Full
6 - Full
7 - Full
8 - Full
9 - 192 + 83 opcodes (though some may exist as internal only functions), 124+16 unused on non-Xmega, and 120 not used anywhere because nothing with that much flash ever existed. So 275 for sure, another 128+16 = 142 for non-xmega, plus if you count opcodes to access memory that has ever existed that's another 120. So 275, 417 or 537.
A - Full
B - Full
C - Full
D - Full
E - Full
F - 1024 opcodes - though 0xFFFF needs to be either a NOP or a skipif like it is now, otherwise empty flash will do unpredictable things, which is a bad thing because that's where code ends up after a wild pointer is called.
```
There are 1024 opcodes in the 0xF8-FF range, half of that range, the range where the low byte is > 0x7F, and 1023 of them could in theory be used
1023 + 255 + 275 = 1553 conservatively, 1695 more generously (stealing the xMega ones), or 1815 most generally (taking ones only applicable to flash sizes nobody expects to ever see. I know I am expecting, at most, 256k flash on a modern AVR, never more than that), open opcodes. The instruction space is 96.93 - 97.58% used, and what little space there is fragmented.

Of course, that all is ignoring the possibility that there are other instructions used during the testing process not documented (these would most likely be located in 0x94-95, amongst those fragmented oddball instructions.

An interesting experiment might be to use means other than the compiler to construct hex files that monitor the system state (which could be constructed with the compiler) as the invalid opcodes are used systematically to see if any of them do anything unexpected.

It is worth noting that the LowI/O instructions (SBI, CBI, SBIC, SBIS) need 8 opcodes each per register, and with 4 registers per port, that means it would take 128 opcodes (plus compiler support)to give the magic of low I/O registers per per fully capable port added. A 100 pin modern successor to the 2560 could avoid the existence of "second class pins" by using 128 opcodes (in addition to whatever else they need to do to make that work) per port. If it followed the example of the 2560, that would probably mean 88 port pins, up from, the 56 we have now. That's only 32 pins or 4 ports. And coincidentally... the first half of the high I/O space is currently totally unused, that could be done, in theory.

There are 1024 opcodes in the 0xF8-FF range, half of that range, the range where the low byte is > 0x7F.
1024 + 255 + 275 = 1580 conservatively, or 1696 more generously, open opcodes. The instruction space is 96.93 - 97.58% used, and what little space there is fragmented.
and that all is ignoring the possibility that there are other instructions used during the testing process not documented (these would most likely be located in 0x94-95, amongst those fragmented oddball instructions
