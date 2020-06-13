# Best Practices for AVRs in Arduino

THIS GUIDE IS IN VERY EARLY STAGES

Poor programming practices are endemic to the Arduino community, and result in poor sketch performance, wasted flash/RAM, difficulty porting sketches to other parts. These can also make your code difficult for others to understand, for example if you post on the Arduino forus requesting assitance. These can also increase the liklihood of bugs that result in the MCU crashing or resetting, as well. For best results, read this guide **BEFORE** you write the sketch (this is not intended as a debugging guide, but rather a set of practices to reduce the need for debugging). 

It is recommended that all design be performed targeting *chips* not *boards* - this eases the process of moving from a general purpose development board (like an Arduino board) to a custom produced board for your application - unless you're sure that your design will be a one-off, don't design to the specific quirks of the development board you happen to have.

This guide applies to all AVR parts supported by Arduino via official board packages or third party hardware packages. (if you are the author of a third party hardware package that supports parts not listed, and want to be included on this list, please contact me). This does not cover the 16-bit XMEGA processors.

## Terminology
This guide uses two unofficial names to refer to the two broad classes of parts to which this guide applies. This is not the official terminology that Microchip uses. 
* `classic AVR` refers to any AVR part with the old syle peripherals - specifically, each port is controlled by registers named PORTx, PINx, DDRx (where x is the letter representing the port in question). The classic AVRs have very similar peripherals - they basically have a set of peripherals chosen from a list of standard peripherals (though there are a few parts with one or more unique peripherals - for example, the tiny x5 and x61 series both have a "high-speed timer" instead of the usual Timer1, though there are significant differences between the timers on those parts). See [classicperipheraltable.md](this table of Classic AVR peripherals and which parts have them) for a full list (this table has not been created as of 4/14/2020, but it will be)
* `modern AVR` refers to any AVR part with the new-style peripherals - this currently includes the tinyAVR 0-series and 1-series, the megaAVR 0-series, and the AVR-DA series (which appears to be what would have been the megaAVR 1-series, had they not changed the branding. This had previously been referred to as "megaavr" in many Arduino circles, amd that is what the IDE internally calls this architecture - however this terminology is unofficial, and presents confusion, since Microchip refers to all ATmega processors as "megaAVR". We try to avoid using `modern AVR` except where it is necessary.

The official terminology for these series of parts is as follows:
'tinyAVR` - All 'ATtiny' devices.

`tinyAVR 0-series` and `tinyAVR 1-series` - The ATtiny devices featuring the new peripherals

`megaAVR` - All 'ATmega' devices

`megaAVR 0-series` - The ATmega4809, 4808, and smaller-flash versions of these parts, featuring the new peripherals.

`AVR DA-series` - The newest 8-bit AVR devices, featuring the new peripherals with enhanced features.

Additional terminology
`mxxx` and `txxx` are shorthand for ATmegaxxx, eg, m328p refers to the ATmega328p, t3216 refers to the ATtiny3216. This shorthand is derived from the abbreviations used by AVRdude to refer to these parts. `tnxxx` is also sometimes used for the ATtiny parts, though less commonly; this is the abbreviation used by the compiler toolchain, though rarely seen outside of it.

`toolchain` refers to the package of compiler tools and libraries required to compile code for a given architecture. In the context of Arduino and AVR parts, this consists of avr-gcc, the device-specs which contain information on specific parts, the linker scripts which tell the linker where to locate the compiled code within the compiled binary, the precompiled .a and .o files for specific parts, and avr-libc, the collection of standard C/C++ libraries that provide basic functionality for the AVR parts and provide very basic wrappers around AVR peripherals. avr-gcc and avr-libc are available as separate packages, while the part specific libraries, including io*.h are supplied by the "atpacks" provided by Microchip. The Arduino compiler toolchain packages contain all three, and precompile a large number of device-specific .o files.

`core` colloquially refers to the package of files that can be used to add support for new parts to the IDE; that is why almost every hardware package includes "core" in it's name. Technically, the "core" specifically refers to the implementation of the Arduino functions on that class of processors, rather than the rest of the package - the board, platform, programmer, etc definitions. Depending on the specific hardware package (and the demands of the parts it supports), the core may or may not be shared with other hardware packages (for example, almost all classic megaAVR parts 

`UART` - Also called a serial port - this is what is used by Serial. The name stands for Universal Asynchronous Receiver-Transmitter. 

`USART` - As above, but has an additional feature that an XCK pin can be used as a clock, instead of using a pre-arranged baud rate. It is common for `UART` to be used to refer to either type, since a USART can act as a UART (though the inverse is not done, and would be inaccurate, since a UART cannot do synchronous serial, only async). Stands for Universal Synchronous/Asynchronous Receiver-Transmitter. AVR parts generally have a USART, not just a UART - however, use of the XCK pin is unheardof in Arduino applications, and quite rare in general - it is a "low cost" feature for the manufacturer to implement, and goes alongside support for reconfiguring a USART to act as an SPI port, which is supported by AVR parts.

## Basics
* Use F() macro for string constants printed with Serial.print() - even though this is a no-op on 0-series and 1-series parts, it imposes no cost there, and makes it easier to port code to classic AVR and DA-series parts.
  * Don't depend on getting `_flashStringConstant` back from F()
  * Don't use PROGMEM on modern AVRs - anything declared const will be kept in flash, yet can be accessed like a normal variable (yay for memory mapping!)
  * if you think your code might end up running on either classic or modern AVRs, use #ifdefs to select the right implementation. (insert example here)
* Use Blink-Without-Delay techniques and millis() for timing
* Use fast digitalRead/Write/pinMode where possible

## Less-basics
* pulseIn() kind of sucks - particularly on modern AVRs, you can do way better with input capture and a type B timer (see [InputCapture example](InputCapture.md) - for one thing input capture is non-blocking if you want it to be (that is, you can do it in the background while also doing something else)
* 

## How to refer to pins
While all of these packages do provide "Arduino Pin Numbers" and "Analog Pin Numbers" (eg, pins named "A0" and such) it is recommended that you refer to pins using their "port pin numbers" PIN_Pxn notation (ie, PIN_PA2 refers to PA2, pin 2 on PORT A). These are immune to confusion relating to different pin mappings (which are common, for historical reasons, and provide maximum portability between different parts in the same families (particularly with the newer parts, where devices with different pin counts from the same family will typically have the same functionality on the same port pins. 

You **cannot** refer to these with "PIN_xn" (ex PIN_B2). 
Some older cores do not support the PIN_Pxn notation; see the above recommendations of hardware packages.


