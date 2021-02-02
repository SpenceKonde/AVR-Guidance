# Reading hex files
Reading hex files by hand is terrible. Try to do everything you can to avoid it! 

If you really do have a hex file that contains valuable information for some reason, and you need to figure out something about it... or maybe modify it:
1. Make a backup
2. Load up something that can do regexs substitutions (I use sublime text and love it)

Maybe it's some compiled piece of proprietary - yet free - software, and it would be just perfect, except that it makes one bad assumpting about how pins will be connected. Maybe it, oh, uses the analog voltage reference - maybe it uses the analog voltage reference pin, while there's a dirt cheap breakout board for the e VREF pin broken out...
Well - without actually learning any fancy tools, one (*can* manually ("mentally execute") a hex file. It's unpleasant, and slow, and tedious, but sometimes, you've gotta do what you've gotta do. 

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

0    0000 0000 0000 0000 nop
0    0000 0000 xxxx yyyy unused except
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
9    1001 010k kkkk 110k jmp (>128k flash, followed by address)
940E 1001 0100 0000 1110 call (<128k flash, followed by address)
9    1001 010k kkkk 111k call (>128k flash, followed by address)

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
~
9F   1001 11rd dddd rrrr mul   

A    10q0 qq0r rrrr bqqq ldd
A    10q0 qq1r rrrr bqqq std

B    1011 0PPd dddd PPPP in
B    1011 1PPr rrrr PPPP out

C    1100 LLLL LLLL LLLL rjmp
D    1101 LLLL LLLL LLLL rcall
E    1110 KKKK dddd KKKK ldi


F    1111 00ll llll lsss brbs
F    1111 01ll llll lsss brbc

F    1111 100d dddd 0sss bld
F    1111 101d dddd 0sss bst
F    1111 110r rrrr 0sss sbrc
F    1111 111r rrrr 0sss sbrs

```

The distribution/usage of opcodes is interesting though: 
* 3/8ths of the instruction-space is taken up with `xxxx-immediate` instructions! (ever wonder why only work on the top 16 registers? Now you know - not enough bits in a 16-bit instruction to let these take up any more of the instruction-space than they already do
* 1/8th is load/set with displacement (well, if you were ever wondering why they don't have an X-register version of those, there's your answer! 
* 3/16ths or so math and logical/arithmatic operations.
* rjmp and rcall are each 1/16th of the instruction-space, and in/out is another 16th. 
* 1/32nd conditional branch, 1/32nd skip-if

Do you think this whole thing is silly and impractical? That it's not good for anything except playing armchair-instruction-set-engineer?
Remember the bit about `lds` and `sts`being particularly useful?
That's how https://github.com/MCUdude/microUPDIcore/commit/49878acda00474831b03005a3cebbdd498bb4b8c#diff-299fd66b54cb0c05b308b3969047fdd0e17f132cedfd167008fb192dc303a19e happened!

("well, there's only one register on the whole chip that controls whether AREF is used... now that we reformatted the hex so we can read it, let's search for the address - oh, here, 1 match! And sure enough, that's an STS immediately before it, storing r16 to it... uhoh, what value is in r16? could be anything, and how could we clear the key bit without breaking something else? I dunno how to do that, but what's the instruction right before that one? first digit s E, thats an ldi, third digit a zero - oh lookit that, it's an ldi into r16!")

