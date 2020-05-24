# Hardware and Board Package selection
Try to choose the right board package for your hardware from the start - particularly with classic AVRs, some of the older cores have weird non-standard pinouts (which is also why, for a lot of the long-popular parts, the recommended cores have multiple pinout options)

## Use the right hardware package
For best results, use the suggested hardware packages below. The official Arduino board packages are usually not suitable for advanced projects, unless your hardware configuration is the same as the official ones, as they are designed to target specific boards, with clock frequency and other design decisions matching only that particular board. If you're using an offical board, go ahead and use them. Parts not listed below are not recommended for new designs. While hardware packages that support them may be available, their use is not advisable in new designs (please PR/create issue if there are good parts that are not listed below). 

Note that currently, devices with native USB are largely beyond the scope of this guide (and my experience) - and as far as I am aware, there is only one choice with Arduino support amyway, the 32u4 - there just aren't many AVRs with USB support in the first place. vUSB is a whole diferent animal It's pretty neat that they managed to implement bitbang USB though, and in a tiny amount of flash, and native USB is pretty cool... but it's also a little trickier to work with (some day I need to look into this - I wonder if I could convert it to a library that could be pulled in for my cores? I have the advantage in that I can add in support on the core side too) 

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
Early on in a project, you have to pick what microcontroller you're going to use. As long as you're not making a custom board (yet), it's not set in stone - you can move up to a more capable one with more pins, or more memory. It could be as simple as changing some pin assignements, oe more complicated (if you used a lot of registers or peripherals directly - though moving these between classic AVRs or between modern AVRs). Once you're on a custom made board, the cost to switch if you discover that the chip you put on the boards doesn't cut it is a lot higher - not only in money, but also the time it takes to get a new batch made. You should always expect that during the course of the project, you'll find a reason you need a few more pins - preempt that by making sure you have a few unused pins for when that happens. 

## Include a programming header!
* ISP for classic AVRs, power+ground+UPDI for ones that use UPDI
* For UPDI parts, either use the Atmel/Microchip 2x3 pin connector pinout, or a 3-pin header with ground in the middle (if you have a series resistor on the UPDI pin (even a few hundred ohms is fine)) the connector can be reversed without damage - if ground is not in the middle of a 1x3 header, reversing the connector would swap power and ground, and applying reverse polarity will damage the device. The protection diodes on these modern AVRs have unreal specs as long as Vcc < 4.9v, so having power connected to UPDI via series resistor is safe. 
* Yes, do this even if you expect to program the devices befor installing them - you will never regret it, and will probably end up being thankful for it down the line!

#### General considerations

**For new designs, the tinyAVR 0-series and 1-series, which have the new peripherals are recommended** - they are cheaper, have better peripherals, and are the future of the AVR product line. However, the library support for these parts has still not caught up; be sure to check that any libraries you plan to use support these parts. A good "quick test" would be to attempt to compile a library example with the part you're considering selected. Errors of the form `'variable' was not defined in this scope` where variable is a short combination of capital letters (ex "TCCR1A") are indicative of this - those are names of hardware registers, which are not present on all parts. You will also find that there are some incompatibilities among classic parts of this sort - different parts have different peripherals, and some name them different things for historical reasons. Some libraries only support a specific subset of parts and will #error if they see an unsupported parts (I recall a cases where library would run without modification on most classic AVR devices - except that it tested for an ATmega2560, ATmega1280, or ATmega328p (and maybe the 328 and 168), or the ATtiny85 (where I think it did do some things differently) - and if wasn't onw one of those parts, it would error out. Which is an example of poor practice in library code!)

Make sure you have as many serial (USART) ports as you need. Software Serial Sucks, try not to have to use it! 

If you, for some reason, need both I2C slave AND master mode, use either one of the few classic AVR devices with two I2C interfaces (I think this is just the PB parts), or a modern 

#### Specific considerations

**If you're looking at using the basic ATmega328p, consider the ATmega328pb** - it has more usable pins, more timers (hence more PWM channels), and an extra serial port, and it costs less, at least from major US disributors. 

**The LGT (LogicGreen) atmega328p-nearly-clone-but-better has great hardware** but the core is a heinous hackjob that doesn't take full advantage of the hardware, and documentation is still pretty poor. It's also unclear where 
