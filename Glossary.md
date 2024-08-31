# Glossary of terms
This covers terms which are not already in ubiquitous use within the Arduino and AVR communities in the wider internet. It came out of a need for words or phrases to describe certain concepts which are routinely referenced. In some cases these terms were in use - although not common use - before. In other cases, descriptive words were chosen in the hopes of being self-explanatory (but which may not be). Confusing terminology used in the documentation for cores that I maintain should be covered here - Please report as an issue for this repo if you find words that need a definition in my documentation.

**Status: Far from complete, but still useful**

## Terms for types of AVRs
Docs I write use two unofficial names to refer to the two broad classes of parts to which this guide applies. This is not the official terminology that Microchip uses. 

### `classic AVR`
Any AVR part with the old-style peripherals - specifically, each port is controlled by registers named PORTx, PINx, DDRx (where x is the letter representing the port in question). The classic AVRs have similar peripherals - they basically have a set of peripherals chosen from a list of standard peripherals (though there are a few parts with one or more unique peripherals - for example, the tiny x5 and x61 series both have a "high-speed timer" instead of the usual Timer1 - and these high speed timers are not the same though they share some features, there are many that they don't share. The same phenomenon is seen in other specialized peripherals. All AVRs released prior to 2016 are classic AVRs. 

### `modern AVR` 
Any AVR part with the new-style peripherals. All parts released since 2016 are considered modern AVRs for this purpose - this currently includes the tinyAVR 0/1/2-series, the megaAVR 0-series, and the AVR-Dx series (which appears to be what would have been the megaAVR 1-series, had they not changed the branding). This is referred to as "megaavr" in many Arduino circles, and that is what the IDE internally calls this architecture - however this terminology is unofficial, and presents confusion, since Microchip refers to all ATmega processors as "megaAVR", so an ATmega328p is a "megaAVR" to Microchip, but not a "megaavr" to Arduino. While an ATtiny412 is a "megaavr" to arduino, but most certainly not a "megaAVR" to Microchip!

### official terminology for these series of parts is as follows:
* `tinyAVR` - All 'ATtiny' devices.
* `tinyAVR 0-series`, `tinyAVR 1-series` and `tinyAVR 2-series` - different product lines with different sets of peripherals. The 0 and 1-series came out at the same time, with the 0-series being a less capable, cheaper product line, while the 1-series had several additional peripherals (some of which would not appear again until the AVR Dx-series). The 2-series dropped many of the premium features of the 1-series in favor of a real differential ADC with improved resolution, programmable gain amplifier and a greatly improved accumulation feature. These parts are numbered via a consistent and logical scheme: The 3 or 4 digit number starts with the flash size in kb (1 or 2 digits, always a power of 2), followed by the number of the tinyAVR series (0, 1, or 2), and finally a pincount identifier: 2 indicates an 8 pin part, 4 a 14-pin one, 6 a 20-pin one and 7 a 24-pin one. An "ATtiny3226" thus has 32k of flash, the fancy ADC, and 20 pins. 
* `megaAVR` - All 'ATmega' devices, most of wehich are classic AVRs. These are identified by a number starting with the flash size (in kb), then one or more numbers to indicate featureset and pincount, sometimes a P (indicating lower power sleep modes), and sometimes an A or B (indicating a major revision of the part took place; the later version being typically intended as a near-drop-in replacement. If the tens-digit is a 0, it is a modern megaAVR 0-series part. Others are all classic AVRs. 
* `megaAVR 0-series` - The ATmega4809, 4808, and smaller-flash versions of these parts, featuring the new peripherals.
* `AVR DA-series`, `AVR DB-series` and so on, featuring the new peripherals plus additional enhanced features. It looks like Microchip at this point realized that they weren't going to have enough decimal digits to represent the full range of pincounts they wanted to use, and they had to adopt a new system for naming them. The part numbers are of the form `AVR___Dx___ where the first blank is the flash size in kb, the x is replaced with a capital letter denoting the feature-set, and the final blank is filled in with the pincount (as a - thus far - 2 digit number). 
* `AVR Dx-series` - The DA-series, DB-series, DD series as a whole. I consider this official as this is in the Microchip ATPACK name.
* `AVR Ex-weries` - An upcoming product line that will start with the EA-series, which has the advanced ADC that first appeared on the tinyAVR 2-series. Curiously, it appears to also be targeting a somewhat lower end of the market - pincounts range only from 28 to 48. and flash ranges from 8k to 64k, the clock speeds are lower than the Dx (it seems to have the clock subsystem that the tinyAVR parts have, rather than the improved one on the Dx), there's no type D timer, and so on.

### Colloquial and unofficial terms
#### `Specimen`
Any single specific part - references to a "part" or "device" imply "any device of that model number", and thankfully if we have a pile of 10 AVR128DB48s in front of us, it shouldn't matter which one we pick up. Occasionally it does: maybe we're hoping to overclock it, in which case the (unmarked except on tinyAVR 0/1-series) temperature grade matters (the high temp one overclocks better). Or maybe we're doing stupid things like setting up an even CCL with the synchronizer turned off, inputs 1 and 2 masked, input 0 set to feedback, TRUTH set to 0x01 (if input 0 is 0, output 1, otherwise output 0. Since the input is it's own output, and this is fully asynchronous, it will sit there oscillating as fast as it can, which is much faster than the CPU can run and you'll need to turn off the bandwidth limiter on your scope to see this. This frequency is highly dependent on both environmental conditions and varies among specimens. (I suspect it is correlated with overclockabililty, which hold out hope of an "overclockability" test that will provide a more definitive test than "Well this calculation ran repeatedly for two days at 5V and 40 MHz and 27C according to the temp sensor... so I guess it's limits lie somewhere beyond there" - more like "The CCL self-oscillates at a speed about X times that of the maximum speed that the ALU should be expecteed to run at stable up to (some fraction) of the self oscillation rate of any specimen - if we could outline such a calculation, that would be of great use to anyone hoping to get a few extra clock cycles. 

#### `Genus`
An adhoc term used to refer to the the different liniages of AVR processors. No universal definition, beyond that parts with MPC (manufacturer product code, also what the 32-byte SIB returns) starting with 'KV' (the Dx-series) are clearly a different genus than parts where the MPC starts with '59' (modern tinyAVRs, modern megaAVRs, and Ex-series). The KV parts run faster (only 20% faster headline numbers, but they have no voltage speed grades since the core runs from an internal regulator and is always running at the same V<sub>core</sub> (1.9V >= V<sub>core</sub> >= 1.6V is suspected, see DB elect. characteristics graphs, power consumption for almost all peripherals shows the hiccup as power switches over from direct to regulated). And the KV/Dx-series can be clocked sufficiently faster. The '59' appears to go up to at most maybe 32 MHz at 5.0V. The 'KV' will do 32 from internal osc without tuning or anything, and will run at 40 or even 48 (for E-spec parts) if it supports a crystal. I wonder what the origin story behind them is - my pet theory is that the modern AVR peripherals were made by taking pieces from classic and xmega parts and trying to put them together in a more coherent manner woth a major emphacis on banishing dumb features cost more in complexity than the functionality is worth while two teams worked on implementing AVRxt - both with the same spec in hand, one of them started with AVRe+ and made AVRxt by building upon that, releasing the tinyAVR 0/1 series  while the other started with AVRxm and created AVRxt starting from the top creating the Dx-series.

## General Terminology
Sorted by approximate frequency of use

### `target` 
the device being programmed. Generally used when the programming is performed through another device, referred to as the `programmer` - particularly when the programmer is also programmable via the Arduino IDE (such as with Arduino as ISP, or jtag2updi) and probably has pins with the same or similar names as the target.

### `toolchain`
The "toolchain" refers to the package of compiler tools and libraries required to compile code for a given architecture. In embedded systems there are two sides to this - the build tools themselves, which compile C to the instruction set in question (AVR here, as this is the AVR guidance repo), and some scheme for providing specific information about the exact part being programmd. In the context of Arduino and AVR parts, this consists of avr-gcc, the device-specs which contain information on specific parts, the linker scripts which tell the linker where to locate the compiled code within the compiled binary, the precompiled .a and .o files for specific parts, and avr-libc, the collection of standard C/C++ libraries that provide basic functionality for the AVR parts and provide very basic wrappers around AVR peripherals. avr-gcc and avr-libc are available as separate packages, while the part specific libraries, including io*.h are supplied by the "atpacks" provided by Microchip. The Arduino compiler toolchain packages contain all three, and precompile a large number of device-specific .o files.

### SFR
Abbreviation for Special Function Register, the techically correct term for what is colloquially called a "register". These control the functioning of on-chip peripherals like timers, serial interfaces, and many other aspects of the operation of the chip. On AVRs, they reside in the "data space", the same address space as RAM, and can be accessed in the same way - except that what is written to them, or even the mere act of reading or writing them, may have arbitrary side-effects. Reading the data register for an interface may clear the interrupt flag indicating that there is new data to read, for example. SFRs often have some "reserved" bits; these usually either have no effect. Sometimes, but not always, they also read as a 0 even if you just wrote a 1 to them - values don't "stick". Usually multi-bit fields in SFRs are "sticky", but with the "reserved" options either duplicating another option (FRQSEL = 0x0C-0x0F) or less often, result in total loss of function on the relevant peripheral (setting the count prescaler to TCD0 to the reserved value will do this, for example). Occasionally they do something different (or example, FRQSEL = 0x0A and 0x0B give internal oscillator clocks of 28 and 32 MHz respectively on Dx - in excess of rated maximum clock speed, but I've never seen them fail at room temperature. The PLL has a secret 4x option, the analog comparator has a secret low power profile which I suspect turned out to not work. 

#### Recognizing SFR names

**On classic AVRs**, names for both SFRs and bits within them are short sequences of capital letters and numbers (typically an abbreviation that makes sense in the context of the peripheral). The names of bitfields refer to their bit position, ie, to set the BAR bit of SFR FOO, you would do something like `FOO |= (1 << BAR)`

**On modern AVRs**, names for SFRs take the form of PERIPHERALn.REGISTER (flat form with _ insead of . works too, and can be used in some places where the other form would be problematic), The represention of bits and bitfields is also richer. Single bits are given in two forms: `PERIPHERAL_BIT_bp` (bit position - 0-7) or `PERIPHERAL_BIT_bm` (bit mask - 1 << (0-7)) (ex: `FOO2.BAR &= ~FOO_BAZ_EN_bm` is equivalent to `FOO2.BAR &= ~(1 << FOO_BAZ_EN_bp)`. Multibit bitfields come with a a `gm` (group mask), and `gp` (group position) macro #defined, and each option is provided as `PERIPHERAL_BITFIELD_OPTION_gc` (group codes). The last of those are provided as enumerated types (enums), so you cant test for them in #if statements. 

#### How to work with SFRs. 
So SFRs behave much like volatile variables (almost always uint8_t) with specific addresses... except thet they also control peripherals and/or controlled by peripherals. Changing their values changes the behavior of some peripheral in many cases; in other cases they are updated by both the peripheral and software (often in this case, on AVR, the hardware sets interrupt flags, and the software clears them) and in still other cases, they (or even just some portion of the bits in an SFR) are read only, but contain status information from the peripheral. They may have side effects when read or written (reading from serial interface data registers advances a 2 byte FIFO, writing to them in many cases causes data to be sentSome also have special behavior when written. Interrupt flags are generally backwards (write 1 to clear the bit, 0 to leave as is, PERIPH.INTFLAGS = bitmask will result in PERIPH.INTFLAGS being set to (PERIPH.INTFLAGS & (~bitmask)), (though it s- this is desirable, as you only heve to specify the bit you want to clear, not which bits you want to preserve), a few bits are "strobes" and do something but don't chenge value (like the software event strobes) when written. Whenever there are sidewffects to reading or writing a register, or bits behave abnormally in ways more complicated than merely being read-only, it  is clearly stated in the datasheet under "register description". Read-onlyness shown by the R or R/W under each bit at the top of a registers entry in the register description section of the corresponding datasheet 

On modern AVRs (and classic ones - but without a uniform procedure for doing so) there may be a special "timed write" procedure to write to SFRs that are considered "important". Use `_PROTECTED_WRITE()` to write to them (this will handle writing the CCP register and ensuring that the write to the target SFR happens in time). 
* Software Reset, Watchdog Timer, CPUINT configuration options that change big-picture interrupt handling behavior. 
* Some seemingly innocuous parts of TCD like turning an output on or off - it's envisioned to be used to control things where misbehavior would result hazards to life and safety. Thing high power electric converters that could have preposterously high short circuit current, BLDC motors that driven improperly can tear themselves apart, and so on, so they provide means to be extremely careful about it. For everyone else, it's a real pain in the ass.
* Flash Mapping and write/read protection
* Self-programming (use `_PROTECTED_WRITE_SPM()`)

Real Life Examples:
```
// configuring the ADC prescaller, referece, and sample capacitor value on a non-2-series tiny/Ex-series part
ADC0.CTRLC  = ADC_PRESC_DIV32_gc | ADC_REFSEL_VDDREF_gc | ADC_SAMPCAP_bm;

// Setting the PGA tunables on the 2-series tinies - using the flat names. 
ADC0_PGACTRL = ADC_PGABIASSEL_3_4X_gc | ADC_ADCPGASAMPDUR_15CLK_gc;

//from init_clock() of megaTinyCore to enable the CPU clock prescaler and choose 16x division (for 1 MHz operation, assuming 16 MHz base clock speed)
_PROTECTED_WRITE(CLKCTRL_MCLKCTRLB, (CLKCTRL_PEN_bm | CLKCTRL_PDIV_16X_gc));

// erase the flash page and write the current page buffer on a tinyAVR
_PROTECTED_WRITE_SPM(NVMCTRL_CTRLA,NVMCTRL_CMD_PAGEERASEWRITE_gc); 

// enable flash writes on a DX-series parts (There are other commands that put it into page erase mode or multipage erase mode and so on - but to actually write or erase anything after doing this, you subsequently write to flash address with SPM (1 word at a time) or ST (1 byte at a time).
// In another example of how certain SFRs can be weird, to change the current command on a Dx, you need to set it to 0 and then to the command you want it, with a protected write each time - otherwise it will reject the command, and NVMCTRL.STATUS will indicate an error. 
_PROTECTED_WRITE_SPM(NVMCTRL_CTRLA,NVMCTRL_CMD_NOCMD_gc); 
_PROTECTED_WRITE_SPM(NVMCTRL_CTRLA,NVMCTRL_CMD_FLWR_gc); 
```
### Register
The word register is a somewhat troublesome word, as it has myriad meanings, which all refer to (usually) a byte of memory that is special in some way.

**In the context of C/C++**, register usually refers to an SFR as described above. 

**When communicating with an external device**, documentation will often refer to memory locations in that device's memory which have special functions (analogous to SFRs on the part you're programming) as "registers", and describe how to write to them over whatever interface you have at hand.

**Finally, in the context of assembly**, "register" can refer to one "working registers"; AVRs have 32 8-bit working registers, numbered r0 through r31, often used in pairs. All of the data that is used in calculations must first be loaded into a working register. Across all scales, it is common for computers to have several types of memory on a continuum from very fast, very small memories to larger slower ones. Generally the fastest type of memory is called a working register. Tightly intertwined with the underpinings of the architecture. The numeric opcodes that correspond to an 


### `UART` 
Also called a `serial port` - this is what is used by Serial. The name stands for Universal Asynchronous Receiver-Transmitter. See also `USART` below for usage notes. 

### `USART`
As above, but has an additional feature that an XCK pin can be used as a clock, instead of using a pre-arranged baud rate. It is common for `UART` to be used to refer to either type: a `USART` can act as a UART, but a `UART` cannot do synchronous serial, only async. Stands for Universal Synchronous/Asynchronous Receiver-Transmitter. AVR parts generally (all?) have a `USART`, not just a `UART` - however, use of the XCK pin is unheardof in Arduino applications, and quite rare in general - it is a "low cost" feature for the manufacturer to implement - likely basically free if there is support for reconfiguring a `USART` to act as an SPI port, which is ubiquitous on AVR parts.

### `mxxx` 
Where the x's are numbers and sometimes letters - common abbreviation for megaAVR devices. Refers to an `ATmega xxx. Often used by programming tools. 

### `tnxxx` and `txxx`
As above, but for tinyAVR parts. The convention of tnxxx comes from the naming of the io header files that Microchip supplies, ex, the ATtiny3224 might be abbreviated t3224 in informal discussion or where space is at a premium. The header file that contains names for all the registers on that part is named iotn3224.h. 

## General programming Terminology
###`blocking` vs `non-blocking` 
A "blocking" function, call, or routine is one which, once called, will not return until some task has completed - when a function is described in this way, it implies that the function, internally, is likely spending time in a "busy-wait" state, or that it is waiting on a potentially very slow external event. . The most familiar example is the use of delay() for timekeeping vs millis(). delay() is an example of a blocking call - Like most blocking functions, it is easier and simpler to use than the non-blocking versions, but it's blocking nature make it frequently unsuitable. Non-blocking things (such as Serial prints from hardware serial ports) rely on interrupt driven stuff happening in the background. In addition to the obvious, there are tricky implications like this:
```
// mySerial is a SoftwareSerial instance; writes are blocking. Serial is a harware serial port, and writes are not blocking. 
mySerial.begin(9600);
mySerial.println("hello world!"); // this call takes - 14 characters (including the cr/lf combo) times 10 bits = 144/9600th's of a second to return. 
digitalWriteFast(PIN_PA7, HIGH); // Pin will be written only after the linefeed is output.

Serial.begin(9600);
Serial.println("hello world!"); // This call stuffs the string into the buffer, turns on the DRE interrupt, and returns. 
digitalWriteFast(PIN_PA7, HIGH); // Pin will be written part-way through the "h"!
// after another 144/9600ths of a second, the ISR will have been called 11-13 times (9 on modern AVRs - there's a 2 byte FIFO
// and the first byte will have been transferred to the shift register; the software ring buffer is only used once those are 
// full, and DREIF is therefor cleared by hardware. 

// but watch out for cases where non-blocking functions become blocking, like this loop: 
// The first few iterations will happen within 1 millisecond. Then the ring buffer is full.
// Now Serial.print() is suddenly blocking waiting for space in the ring buffer, so each 
// iteration takes 20-25ms assuming we're still using 9600 baud.
// When Serial is used for debugging, low baud rates and verbose messages can lead to the
// debugging code changing the behavior of the program dramatically. Make sure you're not 
// printing debug info faster than it can be output. Simply removing the first print will
// make a huge difference, as it's almost 80% of the data being printed! 
// Also, there's no reason to use 9600 baud on a modern AVR running at 16-20 MHz. You can 
// go as high as 921600 baud or 1 mbaud on those if you really need to output that much data,
// and 115200 is a perfectly reasonable speed to default!
for (uint8_t i = 0; i < 100; i++) {
  Serial.print("This is iteration "); // 19 characters
  Serial.print(i); //usually 2 characters 
  Serial.print(' '); // 1 character
  Serial.println(millis()); // assuming this is at start of sketch, 3-6 characters including CRLF - total: 6-9 + 19 = 25-28 characters. 
}

```

### `heap`
Memory allocated using malloc is allocated on the so called heap. As the name implies it is can become messy - and unlike some languages, the resources of the AVR do not make it possible to fix "holes" in the heap when a dynamically allocated chunk of memory is deallocated, but there's another chunk of dynamically allocated memory after it. There's now an obstacle in the middle of the heap, and you can find yourself in a position where there is enough unused SRAM for what you want to allocate - but you can't do it, because there's this other thing in the middle. More powerful processors can host memory management routines of various sorts to clean this up, but AVRs do not have the compute power to do that without unacceptable impacts on performance. One of the worst patterns is repeatedly alternating between realocationng a structure into a slightly larger chunk of memory, and allocating new small ones that get put between them. This happens to be a very common usage pattern with the String class in Arduino, where people add charaters to a strings one at as time. Use char arrays if you can (which are either gloabal, static or local, hence not allocated on the heap), or at least reserve the correct amount of space for the string explicitly (see Arduino String class docs - I have stared at those files for hours and not gained enlightenment regarding their precise functioning.

### `stack`
Memory used for variables with a limited scope (ie, local non-static variables) is allocated "on the stack". The "stack pointer" points to the top of the stack. It starts from the opposite end of the memory as the heap. A "stack-heap collission' is when this dynamically allocated memory runs out, and one of the two gets scribbled over. This is invariably a bad thing, and often diffeicult to debug. Note that in all function calls, the stack containst the address to return to. Accidentally overwriting the contents of the stack, sometimes called "smashimg the stack". particiarly if this happemed not due to a collision but some thing like an out of bounds array access. Regardless of how it happens, it results in unpredictable behavior; most often a "dirty reset" that hangs or resets on a classic AVR or resets on megaTinyCore/DxCore (since the condition is detected and dealt with more gracefully).

### wild pointer
A pointer which points to a nonsensical location, instead of whatever it's supposed to point to; generally the result of memory corruption due to writing outside the bounds of an array or other software mishaps. The term often specifically refers to the case when something incorrect is used as a pointer for ijmp/icall (in C/C++, that means calling a function pointer), resulting in code execution resuming from some wacky place in the program space, though it is also used to refer to a pointer used to access the data space that is pointed to the wrong place. Wild pointers are bad. They result in either corrupted data in memory (or showing bogus data ifit's being read from) when it points to RAM, or having unpredictable effects on an effectively random peripheral if it's an SFR. If the pointer is to a place where the desired data used to be but has been deallocated, it is instead called a "stale" pointer, and if it points to 0 it is a "null pointer", see below.

### Null Pointer
A pointer which points to 0x0000. 
In most architectures, this is always invalid. That is not the case in AVRs. A pointer to 0x0000 is *potentially valid* for access to the data space. On modern AVRs 0x0000 is VPORTA.DIR - perfectly valid (though an odd choice - usually if you've got to use a pointer, you're using the more luxurious facilities of the PORTn registers rather than the VPORTs. On classic AVRs, however, if you're using a null pointer, you *are* definitely doing something wrong. You do not access working registers of the CPU (see Register (in the context of assembly, above) with a pointer. That is absolute madness, and pointing it at r0 is even more insane - on classic AVRs, the first 20 addresses in the data space were the working registers, followed by the I/O space. So 0x01 in the low I/O space was located at 0x21 in the dataspace. That is no longer the case on AVRxt, and working registers aren't mapped to the data space *nor ought they be*. Hence, if you are writing a funtion where a pointer to VPORTA.DIR would make sense, that would be a valid input, and you should not reject null pointers (but you probably have more specific things you can test for). Otherwise, and any time you're on a classic AVR, make sure a pointer isn't null before using it! Be especially sure that you don't call null function pointers. That's the equivalent of call 0x0000, which is an instant dirty reset, that restarts your sketch without an actual hardware reset occurring. 

## Arduino-specific Terminology

### core
Colloquially refers to the package of files that can be used to add support for new parts to the IDE; that is why almost every hardware package includes "core" in it's name. *Technically*, the `core` specifically refers to the implementation of the Arduino functions on that class of processors, rather than the rest of the package - the board, platform, programmer definitions, for example, though the two are usually tightly coupled. Depending on the specific hardware package (and the demands of the parts it supports), the core may or may not be shared with other hardware packages (for example, (almost?) all classic megaAVR parts can use the same core; similarly megaAVR 0-series and tinyAVR 0/1-series devices could use the same core - though currently they do not, for a variety of reasons).

### variant
As noted above, hardware packages will often support many parts which can share the same core. The differences betweem parts with different numbers of pins or port-to-pin mappings are often provided by a single file within the "variants" directory inside the hardware package, which typically contains only a single file: pins_arduino.c. This is the file that contains arrays and macros that map Arduino pin numbers to ports, analog pin numbers (`An` naming) to ADC channels, PWM pins to their timer and channel, and so on. Parts that differ only in the amount of RAM (and sometimes presence or absence of peripherals, for example, tinyAVR 1-series vs 0-series) can be supported with the same variant.

### megaavr
is the name given by the Arduino developers to what was described above as "modern AVR" parts (see also above note on refering to types of parts). It is used in the "architectures" property of libraries to define which platforms are supported by that library, and hence must be followed, however nonsensical it is to have "megaAVR" parts that may not be "megaavr" architecture in arduino terms. 

### Arduino-land
Arduino-land is the Arduino community and the programming practices idioms and styles that are common or widespread there. As a community with a large proportion of hobbyists, many of whom have no programming experience outside of Arduino and high-school age students using it for class, and a relatively small number of professionals using it as a light-weight IDE, method of setting up toolchains and an expedient starting point for development (much of the magic of Arduino, IMHO, is that the same tools and IDE can be used effectively by someone who never wrote code in their life before the start of the semester, and an experienced programmer writing inline assembly, and have it work passably well for both of them. The inhabitants of Arduino-land are diverse and heterogeneous - they run the gamut from complete novice to deep experts, and the community includes many people who are both at once - yours truly has had dreams in AVR machine code... but don't ask me to use any of the features that differentiate C++ from C (At least not if you want me to write working code).

**This is important for you to consider as you write code you hope othes will use and as you interpret advice and read code by others.**

This means that you should expect that both very poor coding practices and very sophisticated code coding practices will be widespread, often within the same program (copy/paste). Ideoms and advice are spread without understanding or fact checking. Cargo cult programming practices are common, and *bad* advice persists far longer than it should. The users who will pick up code that you share may be very inexperienced. Even with my cores, which add support for non-standard parts to Arduino, I routinely receive questions from people who appear to have no experience using common Arduino boards when they pick up exotic new part (or worse, some exotic *old* part) that they want to use. The hobbyists also include a great many retirees from the electronics field, and while they have a great deal of experience, much of it is dated, and they are apt to suggest aging parts because that's what they used back when they were in the industry (I'm sure the TIP102 was hot shit when it was released, but a modern, or even decade old, MOSFET will knock it and other darlington transistors' socks off)`*`, or outmoded approaches to solving common problems. (Occasionally, the resource constraints of embedded systems may make some of those methods reasonable). The U-shaped age distribution, with inexperienced newbies ill-equipped to challenge advice or adapt hardware designs to this era, and retired hobbyists who know exactly how the best way would have been 30 years ago combined with the use of books long after their sell-by date is the reason you may have noticed a preponderance of ancient hardware (BJTs, 7805 regulators, etc) sitting on the same schematic as a couple of snazzy new parts.

`*` Huh? Yeah, transistors totally wear socks. You see that they have legs, generally three, right? Those legs are made out of copper, but look at them! They look silver. That's because the're wearing "socks" - matte tin socks, which help improves solderability.  

When writing code that you plan to share, be aware of the diversity of your audience - but also of the relation between competence and how deeply someone is likely to dig. Novices will generally look to the documentation if they're wise, or crib from some other piece of code or an example if they're not. Experts will sometimes read documentation, but as often as not, will go straight to the code to figure out the answer to their question, assuming that the documentation is crap (which is, unfortunately, often the case with Arduino libraries). Most inhabitants of Arduino-land will treat inline assembly as if it were a magic spell (which it differs from in that magic is capable of intuition and cleverness, while a microcontroller is not, which is at once both a blessing and a curse). Very few of them can look at a few lines of C code and predict what the compiler will do with it, nor look at the compiler output and discern whether the compiler did a good job or not. Most people are shocked and horrified when they see how slowly digitalWrite and similar functions run. 

We would all benefit if there were "annotated datasheets" available, pointing out things like `_PROTECTED_WRITE()` (not mentioned, because that's supplied by avrlibc), features for which there are serious errata, and so on
