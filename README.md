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
`core` colloquially refers to the package of files that can be used to add support for new parts to the IDE; that is why almost every hardware package includes "core" in it's name. Technically, the "core" specifically refers to the implementation of the Arduino functions on that class of processors, rather than the rest of the package - the board, platform, programmer, etc definitions. 
`UART` - Also called a serial port - this is what is used by Serial. The name stands for Universal Asynchronous Receiver-Transmitter.  
`USART` - As above, but has an additional feature that an XCK pin can be used as a clock, instead of using a pre-arranged baud rate. This feature is rarely used, and it is common for `UART` to be used to refer to either type (though the inverse is not done, and would be inaccurate). Stands for Universal Synchronous/Asynchronous Receiver-Transmitter

## Use the right hardware package
For best results, use the suggested hardware packages below. The official Arduino board packages are usually not suitable for advanced projects, unless your hardware configuration is the same as the official ones, as they are designed to target specific boards, with clock frequency and other design decisions matching only that particular board. If you're using an offical board, go ahead and use them. Parts not listed below are not recommended for new designs. While hardware packages that support them may be available, their use is not advisable in new designs (please PR/create issue if there are good parts that are not listed below). 

Note that currently, devices with native USB are largely beyond the scope of this guide - and as far as I am aware, there is only one choice with Arduino support amyway, the 32u4 - there just aren't many AVRs with USB support in the first place. vUSB is a whole diferent animal (though unfortunately it often has compatibility problems 

#### Modern AVR devices
* ATmega4809, 4808 (and smaller-flash versions) incl. Uno WiFi Rev. 2 and Nano Every: https://github.com/MCUdude/MegaCoreX
* ATtiny3217, 3216, 1614, 412 (and 0-series and smaller-flash versions): https://github.com/SpenceKonde/megaTinyCore
* DA-series (AVR128DA) - the newest, top-end parts in the 8-bit AVR line (work is underway for suppoet at this time)

#### Classic AVR devices
* ATmega2560, 2561 (and smaller-flash versions): https://github.com/MCUdude/MegaCore
* ATmega1284p (and smaller-flash versions), incl. Sanguino: https://github.com/MCUdude/MightyCore
* ATmega328/p/pb ( and smaller-flash versions - for new designs, the PB parts are strongly recommended): https://github.com/MCUdude/MiniCore
* ATtiny 84, 85, 4313, 88, 841, 828, 1634, 43, 861, 167 (and smaller-flash versions): https://github.com/SpenceKonde/ATTinyCore
* ATtiny13 https://github.com/MCUdude/MicroCore


## Use the right chip - plan ahead
Early on in a project, you have to pick what microcontroller you're going to use. As long as you're not making a custom board (yet), it's not set in stone - you can move up to a more capable one with more pins, or more memory. It could be as simple as changing some pin assignement, ore more complicated (if you used a lot of registers or peripherals directly). Once you're on a custom made board, the cost to switch if you discover that the chip you put on the boards doesn't cut it is a lot higher - not only in money, but also the time it takes to get a new batch made. You should always expect that during the course of the project, you'll find a reason you need a few more pins - preempt that by making sure you have a few unused pins for when that happens. 

#### General considerations

**For new designs, the "modern" parts with the new peripherals are recommended** - they are cheaper, have better peripherals, and are the future of the AVR product line. However, the library support for these parts has still not caught up; be sure to check that any libraries you plan to use support these parts. A good "quick test" would be to attempt to compile a library example with the part you're considering selected. Errors of the form `'variable' was not defined in this scope` where variable is a short combination of capital letters (ex "TCCR1A") are indicative of this - those are names of hardware registers, which are not present on all parts. You will also find that there are some incompatibilities among classic parts of this sort - different parts have different peripherals, and some name them different things for historical reasons. Some libraries only support a specific subset of parts and will #error if they see an unsupported parts (I recall a cases where library would run without modification on most classic AVR devices - except that it tested for an ATmega2560, ATmega1280, or ATmega328p (and maybe the 328 and 168), or the ATtiny85 (where I think it did do some things differently) - and if wasn't on one of those parts, it would error out. Which is an example of poor practice in library code!)

#### Specific considerations

**If you're looking at using the basic ATmega328p, consider the ATmega328pb** - it has more usable pins, more timers (hence more PWM channels), and an extra serial port, and it costs less, at least from major US disributors. 


## How to refer to pins
While all of these packages do provide "Arduino Pin Numbers" and "Analog Pin Numbers" (eg, pins named "A0" and such) it is recommended that you refer to pins using their "port pin numbers" PIN_Pxn notation (ie, PIN_PA2 refers to PA2, pin 2 on PORT A). These are immune to confusion relating to different pin mappings (which are common, for historical reasons, and provide maximum portability between different parts in the same families (particularly with the newer parts, where devices with different pin counts from the same family will typically have the same functionality on the same port pins. 

You **cannot** refer to these with "PIN_xn" (ex PIN_B2). 
Some older cores do not support the PIN_Pxn notation; see the above recommendations of hardware packages.


