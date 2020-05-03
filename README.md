# Best Practices for AVRs in Arduino
Poor programming practices are endemic to the Arduino community, and result in poor sketch performance, wasted flash/RAM, difficulty porting sketches to other parts. These can also make your code difficult for others to understand, for example if you post on the Arduino forus requesting assitance. These can also increase the liklihood of bugs that result in the MCU crashing or resetting, as well. For best results, read this guide **BEFORE** you write the sketch (this is not intended as a debugging guide, but rather a set of practices to reduce the need for debugging). 

It is recommended that all design be performed targeting *chips* not *boards* - this eases the process of moving from a general purpose development board to a custom produced board for your application - unless you're sure that your design will be a one-off, don't design to the specific quirks of the development board you happen to have.

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

## Use the right hardware package
For best results, use the suggested hardware packages below. Due to dubious decisions made in the official board packages, we regret that we cannot recommend their use. Parts not listed below are not recommended for new designs. While hardware packages that support them may be available, their use is not advisable in new designs (please PR/create issue if there are good parts that are not listed below).

**For new designs, the "modern" parts with the new peripherals are recommended - they are cheaper, have better peripherals, and are the future of the AVR product line** However, the library support for these parts has still not caught up; be sure to check that any libraries used support these parts.

**If you're looking at using the basic ATmega328p, consider the ATmega328pb** - it has more pins, more timers (hence more PWM channels), and an extra serial port, and it costs less, at least from major US disributors.

#### modern AVR devices
* ATmega4809, 4808 (and smaller-flash versions) incl. Uno WiFi Rev. 2 and Nano Every: https://github.com/MCUdude/MegaCoreX
* ATtiny3217, 3216, 1614, 412 (and 0-series and smaller-flash versions): https://github.com/SpenceKonde/megaTinyCore
* DA-series (AVR128DA) - the newest, top-end parts in the 8-bit AVR line (work is underway for suppoet at this time)

#### classic AVR devices
* ATmega2560, 2561 (and smaller-flash versions): https://github.com/MCUdude/MegaCore
* ATmega1284p (and smaller-flash versions), incl. Sanguino: https://github.com/MCUdude/MightyCore
* ATmega328/p/pb ( and smaller-flash versions - for new designs, the PB parts are strongly recommended): https://github.com/MCUdude/MiniCore
* ATtiny 84, 85, 4313, 88, 841, 828, 1634, 43, 861, 167 (and smaller-flash versions): https://github.com/SpenceKonde/ATTinyCore
* ATtiny13 https://github.com/MCUdude/MicroCore

## How to refer to pins
While all of these packages do provide "Arduino Pin Numbers" and "Analog Pin Numbers" (eg, pins named "A0" and such) it is recommended that you refer to pins using their "port pin numbers" PIN_Pxn notation (ie, PIN_PA2 refers to PA2, pin 2 on PORT A). These are immune to confusion relating to different pin mappings, and provide maximum portability between different parts in the same families (particularly with the newer parts, where devices with different pin counts from the same family will typically have the same functionality on the same port pins. 

You **cannot** refer to these with "PIN_xn" (ex PIN_B2). 
Some older cores do not support the PIN_Pxn notation; see the above recommendations of hardware packages.

## 
