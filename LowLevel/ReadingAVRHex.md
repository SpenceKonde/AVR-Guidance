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
00   0000 0000 xxxx yyyy unused except
01   0000 0001 dddd rrrr movw
02   0000 0010 dddd rrrr muls
03   0000 0011 yddd zrrr mulsu/fmul*
0    0000 01rd dddd rrrr cpc
0    0000 10rd dddd rrrr sbc
0    0000 11rd dddd rrrr add lsl

1    0001 00rd dddd rrrr cpse
1    0001 01rd dddd rrrr cp
1    0001 10rd dddd rrrr sub
1    0001 11rd dddd rrrr adc rol

2    0010 00rd dddd rrrr and
2    0010 01rd dddd rrrr eor clr
2    0010 10rd dddd rrrr or
2    0010 11rd dddd rrrr mov


3    0011 KKKK dddd KKKK cpi
4    0100 KKKK dddd KKKK sbci
5    0101 KKKK dddd KKKK subi
6    0110 KKKK dddd KKKK ori
7    0111 KKKK dddd KKKK andi

8    10q0 qq0r rrrr bqqq ldd
8    10q0 qq1r rrrr bqqq std


9    1001 000r rrrr 0000 lds (followed by address)
9    1001 000r rrrr xxyy ld (xx = Z 00, lpm 01, Y 10, X 11)
9    1001 000r rrrr 1111 pop
90, 91 ld/lds/lpm, pop
9    1001 001r rrrr 0000 sts (followed by address)
9    1001 001r rrrr xxyy st (yy = + 01, - 10, 00 for X/lpm only)
9    1001 001r rrrr 1111 push
92, 93 st/sts, push

9409 1001 0100 0000 1001 ijmp
9509 1001 0101 0000 1001 icall 
940C 1001 0100 0000 1100 jmp (<128k flash, followed by address)
9    1001 010k kkkk 110k jmp (>128k flash, followed by address - looks like max is only 512k xmega? So 940D possible on 2560, 941C/941D on 512k parts.
940E 1001 0100 0000 1110 call (<128k flash, followed by address)
9    1001 010k kkkk 111k call (>128k flash, followed by address) - 940F possible on 2560, 941E/941F on 512k parts.

9508 1001 0101 0000 1000 ret
9518 1001 0101 0001 1000 reti
9588 1001 0101 1000 1000 sleep
95A8 1001 0101 1010 1000 wdr
95E8 1001 0101 1110 1000 spm

96   1001 0110 KKdd KKKK adiw
97   1001 0111 KKdd KKKK sbiw

98   1001 1000 pppp psss cbi
99   1001 1001 pppp psss sbic

9A   1001 1010 pppp psss sbi
9B   1001 1011 pppp psss sbis

9C   1001 11rd dddd rrrr mul
9D, 9E, 9F
   

A    10q0 qq0r rrrr bqqq ldd
A    10q0 qq1r rrrr bqqq std

B    1011 0PPd dddd PPPP in
B    1011 1PPr rrrr PPPP out

C    1100 LLLL LLLL LLLL rjmp
D    1101 LLLL LLLL LLLL rcall
E    1110 KKKK dddd KKKK ldi


F    1111 00ll llll lsss brbs
F    1111 01ll llll lsss brbc
all the branch instructions are just special cases of brbs or brbc

F    1111 100d dddd 0sss bld
F    1111 101d dddd 0sss bst
F    1111 110r rrrr 0sss sbrc
F    1111 111r rrrr 0sss sbrs
And these are bit-level access to registers
Also, half of this block is actually unused? Bit 3 could be a 1.
Apparently 0xFF is treated an sbrs though - skip if bit 7 in register 31 is 1? They *seriously* should 
have 'special-case'ed that one to be a nop, so  execution would skid along the 0xFF of empty flash, not 

```

## Okay, I found what needs to change
Some time I'll write about the process of hackign something up in greater detail; key points are:
Try to just change a value. If you can't do that, try to just change similar instructions. 
If you can'd do either of those things, can you tinker with existing control flow  to get what you want? There is an unfortunate possibility that you might need to actually ADD code. Which is particularly painful, but do-able: replace two instructions with a `call` to empty flash, and proceed to act like you're writing a naked ISR. Any register you cannot determine is unneeded at the time of making that call must be preserved. (though, carefuly study wi

The importance of minding the SREG is hard to overstate: all control flow is based on it, with the exception of two families of "skip-ifs" 

## For Armchair ISA Designers
ISA = Instruction Set Artchitecture 

If - like any engineer-type - your first thought is "Huh, those guys made such a bad decision, I could totally do it better" - maybe you even day-dream about how you'd design a better architecture... Well, it's way better to have some familiarity with an example of what you imagine designing, right?

One thing that's interesting to note is the distribution/usage of opcodes (or rather, which instructions eat up huge chunks of the 16-bit number for each instruction)
* 3/8ths of the instruction-space is taken up with `xxxx-immediate` instructions! (ever wonder why only work on the top 16 registers? Now you know - not enough bits in a 16-bit instruction to let these take up any more of the instruction-space than they already do
* 1/8th is load/set with displacement (well, if you were ever wondering why they don't have an X-register version of those, there's your answer! 
* 3/16ths or so math and logical/arithmatic operations.
* rjmp and rcall are each 1/16th of the instruction-space, and in/out is another 16th. 
* 1/32nd conditional branch, 1/32nd skip-if

## Yes, this is viable!

Do you think this whole thing is silly and impractical? That it's not good for anything except playing armchair-instruction-set-engineer?
Remember the opening story of the 32u4 programmer code that's available as hex but not code? That's Microchip's mEDBG. And the bit about `lds` and `sts`being particularly useful?
That's how https://github.com/MCUdude/microUPDIcore/commit/49878acda00474831b03005a3cebbdd498bb4b8c#diff-299fd66b54cb0c05b308b3969047fdd0e17f132cedfd167008fb192dc303a19e happened, and (what immediately prompted it was that, although I consider myself pretty good at soldering - I found the hardware mod to make the stock f/w work to be nearly impossible to achieve. It went down something like this:

"well, there's only one register on the whole chip that controls whether AREF is used... now that we reformatted the hex so we can read it, let's search for the address - oh, here, 1 match! And sure enough, that's an STS immediately before it, storing r16 to it... uhoh, what value is in r16? could be anything, and how could we clear the key bit without breaking something else? I dunno how to do that, but what's the instruction right before that one? first digit s E, thats an ldi, third digit a zero - oh lookit that, it's an ldi into r16! Then, I opened up a clean copy of the file, searching for the swapped-order bytes I needed to change, turned the appropriate 1 into a 0, adjusted the checksum, and Bam! Problem solved!

