# AVR parts vs Arduino core support

Part Family                     | Core supporting | Comments 
--------------------------------|-----------------|--------------------------------
tinyAVR 0/1/2-series            | [megaTinyCore](https://github.com/SpenceKonde/DxCore)| Any ATtiny ending in yz, where y is 0, 1, or 2, and z is 2, 4, 6, or 7 
megaAVR 0-series                | [MegaCoreX](https://github.com/MCUdude/MegaCoreX)| Any ATmega ending in 08 (32-pin) or 09 (48-pin)
AVR DA/DBs-series               | [DxCore](https://github.com/SpenceKonde/DxCore)| AVRxxxDAyy or AVRxxxDByy, where xxx is flash in k and yy is pincount. 
AVR DD-series                   | [DxCore](https://github.com/SpenceKonde/DxCore)| AVRxxDDyy - pincounts of 14, 20, 28 and 32 available
AVR EA-series                   | DxCore planned  | AVRxxEAyy - Unreleased.
Classic tinyAVR w/>= 2k flash   | [ATTinyCore](https://github.com/SpenceKonde/ATTinyCore) | Excludes: t20, t40, t28L
Classic ATtiny13                | [MicroCore](https://github.com/MCUdude/MicroCore) |  
Classic ATtiny15                | None            | Obsolete and out of production
Classic ATtiny20/40             | None            | Crapola parts
Classic ATtiny28L               | None            | Functionally obsolete. And it has no SRAM! 
Reduced Core ATtiny4/5/10/11    | [attiny10core](https://github.com/technoblogy/attiny10core)    | Reduced core parts. Not only are the peripherals disappointing, the CPU is gimped. 
Classic ATmega ending in 8      | [MiniCore](https://github.com/MCUdude/MiniCore)| Like 328p/pb and so on. Used in nano, uno, pro mini. All are better w/MiniCore. 
Classic ATmega ending in 4      | [MightyCore](https://github.com/MCUdude/MightyCore)| Like the 1284p and 324pb.
Classic ATmega16/32/8535        | [MightyCore](https://github.com/MCUdude/MightyCore)| Predecessors to the x4-series
Classic ATmega8515/162          | [MajorCore](https://github.com/MCUdude/MajorCore)| Aging parts with little to recommend them, 
Classic ATmega ending in 0 or 1 | [MegaCore](https://github.com/MCUdude/MegaCore)| ATMega64/128/640/1281/1280/2560/2561. Parts ending in 0 have 100 pins, rest 64.
Classic ATmega ending in 5/50   | [MegaCore](https://github.com/MCUdude/MegaCore)| Combines the pincount of the previous line with inferior peripheral selection.
Classic ATmega ending in 9/90   | [MegaCore](https://github.com/MCUdude/MegaCore)| As m x5/50, only those extra pins can drive arguably obsolete, rarely seen passive LCDs. 
AT90CANxx                       | [MegaCore](https://github.com/MCUdude/MegaCore)| Integrated CANBUS`*`for automotive applications. 
ATmega###RFR/RFR1/etc           | None            | Integrated radio communiction functionality, but the RF suff isn't documented publically.<br/>RF HW design is hard and has legal complications, and the RF functionaliity uses either closed source lib or docs are hidden behind an NDA. 
ATmega16HV/32HVA/HVB/HVrev2     | None            | For controlling LiPo charging up to 4S. Scant documentation. Just buy a balancing charger board. 
ATA (ATAutomotive)              | None            | Little info available, and what is avaiable implies that they are not a good fit for hobbyists. 
AT90_____ not AT90CAN           | None            | Older parts, each family with a confusing peripheral (either USB or a fancy timer similar to modern TCD called a PSC
ATmega16M1/32M1/64M1,32/64C1    | None, sadly     | Never sure why nobody made a core for these. 

`*` CANBUS is a very complicated automotive network. It might as well stand for "Confusing Automotive Network Bus" But it is ubiquitous in automotive use.

## Those ATmega16M1/32M1/64M1/32C1/64C1 parts
This family is the only set of parts that I think "deserves" a core and doesn't have one. The xxM1's absolutely beat the stuffing out of almost every other classic AVR in a 32-pin package, with the exception of the 328P**B**, where experts could argue about the relative merits. Every other classic AVR who would hope to beat out the 64M1 or even 32M1 on any metric will meet humiliating defeat. What makes the lack of a core even more surprising is that most of what is needed is straightforward, not hard to do. I think it would just be a matter of leg-work to get them working. If anyone is interested in doing this, start from MiniCore, adjust variants, tweak wiring_analog to use the slightly fancier ADC - especially if you wanted to implement all the fancy features (it's got differential capability and all that good stuff, see ATTinycore v2.0.0-dev variants for how I suggest handling differential ADC). Some of these parts have CANBUS support as well (I think this can be stolen from the libraries that use CANBUS on the AT90CAN parts, with little or no modification. 
### If you were to change that... 
If someone else were to get most of it working, I would be happy to advise and would even be willing to add the code for PSC PWM via analogWrite (I will have an easier time than most, as I've worked so much with the similar TCD), ADC tweaks, and if you asked nicely, I could even try to make the LIN-USART work in USART mode (for the other things, as long as you're doing the legwork of setting up and maintaining the rest of the core, I'll implement them even if you ask me rudely. The LIN-USART looks little harder.  
