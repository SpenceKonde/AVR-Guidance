# Assembly listing basics (if you don't know avr assembly)
((note: this is written in the contest of the cores I maintain, and the assembly listing & memory map they can generate)

Once the usual sketch-size heuristics have been played out, looking through the assembly listing and memory map is the best technique for figuring out how to reduce flash consumption evem if you don't know any assembly. Start off with the memory map (new since April 2021 releases of my cores). Particularly if you care about code size. The .map file can be opened with any text editor. but you won't like the results. Impoert it into Excel or Google Sheets, it's a delimited text file, and the delimiters are tabs, and the | character. Set the formatting the "text" not number or general - if you let it, the spreadsheet program will trash some of the hexadecimal numbers (the ones thatare made up of only digits 0-9 get converted to numbers, the ones with a-f stay letters. Not good. The column headings - unfortunately - get merged to one cell (I may try to make a script to auitomatically clean the maps after exporting them to fix this. 

That map shows everything, sorted by address with the name or "symbol" listed - in that way you can find functions - though the first thing you will notice is that loop and setup aren't there! That's because in both cases, there is only a single place where they are called - any function that is only called from one place gets "inlined" and shows up as part of the calling function; this is good, because it saves the flash that would otherwise be wasted on the function call overhead, but it does make the memory map less useful (that's why the memory map is only a first step. If trying to reduce sketch size, the size column is of obvious importance. Things that don't have a size listed don't make it into the binary. Look for large numbers in the size column. At the bottom, the addresses will suddenly jumop into the 0080000 range. Addresses after 00800000 are in RAM not flash - though if it says they are in data, they do also show up in flash ("rodata" is read-only data stored in flash but not RAM on parts with mapped flash). 

So we may know that there are some particularly suspect functions that look big on the listing. Try to find them in the assembly listing. In the assembly listing, one or more lines of C code are shown, as well as the file and line number they are from, followed by the generated assembly assopciated with them (most of the time - occasionally obj-dump gets confused, and the C source gets "out of sync" with the assembly it generate). Lines of C may be shown in groups, the line number at the top of the group is the number of the *last* line in that group of 

Generated assembly is shown one instruction to a line, and each line will consist of a hexadecimal number, followed by a `: (this is the address of the instruction, which is why they count up 2 at a time)`, then 2 or 4 hexadecimal bytes (this is the numeric value of the instruction), followed by the mneumonic and argument(s) to that instruction, and finally, a; and some semblance of an auto-generated comment. If the instruction jumps or branches to, or calls code elsewhere, the comment will have the address, and the name of the fumnction it's located im. For example the final line below is rjmp and the comment says it points to 0x2bc. And what address is the first line below at? 2bc! since we're looking at loop(), there had damned well better better be a jump at the end pointed to the beginning, right?

```
C:\arduino-1.8.13-portable\portable\sketchbook\sketches\Blink/Blink.ino:33
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);   // turn the LED on (HIGH is the voltage level)
 2bc:   81 e0           ldi r24, 0x01   ; 1
 2be:   0e 94 a9 00     call    0x152   ; 0x152 <digitalWrite.constprop.0>
C:\arduino-1.8.13-portable\portable\sketchbook\sketches\Blink/Blink.ino:34
  delay(1000);                       // wait for a second
 2c2:   0e 94 7f 00     call    0xfe    ; 0xfe <delay.constprop.2>
C:\arduino-1.8.13-portable\portable\sketchbook\sketches\Blink/Blink.ino:35
  digitalWrite(LED_BUILTIN, LOW);    // turn the LED off by making the voltage LOW
 2c6:   80 e0           ldi r24, 0x00   ; 0
 2c8:   0e 94 a9 00     call    0x152   ; 0x152 <digitalWrite.constprop.0>
C:\arduino-1.8.13-portable\portable\sketchbook\sketches\Blink/Blink.ino:36
  delay(1000);                       // wait for a second
 2cc:   0e 94 7f 00     call    0xfe    ; 0xfe <delay.constprop.2>
 2d0:   f5 cf           rjmp    .-22    ; 0x2bc <main+0x7a>
```

This says that line 33 (that first digitalWrite()) creates some assembly instructions, though the 4 above it don't create any. It creates two instructions, at addresses 2bc and 2be, the first one is a single word instruction (*"ldi" whatever that is*), and the second one is a two word instruction, named "call"... and then on to line 34, the delay(), which is at address 2c2; you saw that the `call` was 4 bytes, and 2be + 4 = 2c2 in hex! (*there's that "call" instruction again... and there's hardly anything in here, either... it's as if it's calling code somewhere else to do that stuff... like how we call digitalWrite and delay in C...? Huh... **call**ing?*). 

As you can see, not much happens in loop itself (just like in the C code - this isn't always the case, though, sometimes it will dispense with the `call` and put the function right there) - but if you search for delay and digitalWrite - you can see all the code that gets generated to implement those... and then you see how blink can take up 778 bytes of flash (that's 389 instructions!) 

A less methodical technique to see where the flash goes is to quickly scroll through the file and look for a big blob of assembly instructions in close proximity, then look up from those until you find the name of a function or something recognizable - sometimes that can be hard to figure out if you're new to such low level stuff... but other times, it's bloody obvious. You are hoping to find a place where it's generating tons of code for something that should be simple,  even if you can't understand the assembly, you can tfiddle with that part of the C code knowing where to focus your effort to have a chance of payback. Of course - if you  If you're ever bored, try violating the usual "space saving" heuristics, and examine what changes in the assembly... if you like this, you could also study [the AVR bible](https://ww1.microchip.com/downloads/en/DeviceDoc/AVR-InstructionSet-Manual-DS40002198.pdf)... full of fascinating bits of truth and wisdom from the most authoritative source, phrased in an strange and obtuse manner that you may not understand until days, weeks, or years after you read it. 
