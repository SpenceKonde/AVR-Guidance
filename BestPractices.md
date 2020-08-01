# Best Practices for AVRs in Arduino

## THIS GUIDE IS IN VERY EARLY STAGES
## It should NOT be used as a general guide at this time
**Some pages may be ready for public consumption, and linked to from elsewhere, but as a general guide, it is not**

Poor programming practices are endemic to the Arduino community, and result in poor sketch performance, wasted flash/RAM, and difficulty porting sketches to other parts. These can also make your code difficult for others to understand, for example if you post on the Arduino forums requesting assitance. Poor practices common in Arduino can also increase the liklihood of bugs that result in the MCU crashing or resetting, as well. For best results, read this guide **BEFORE** you write the sketch (this is not intended as a debugging guide, but rather a set of practices to reduce the need for debugging). 

It is recommended that all design be performed targeting *chips* not *boards* - this eases the process of moving from a general purpose development board (like an Arduino board) to a custom produced board for your application - unless you're sure that your design will be a one-off, don't design to the specific quirks of the development board you happen to have.

This guide applies to all AVR parts supported by Arduino via official board packages or third party hardware packages. (if you are the author of a third party hardware package that supports parts not listed, and want to be included on this list, please contact me). This does not cover the XMEGA processors - and to my knowledge they are not supported by any Arduino library.

## Basics
* Use F() macro for string constants printed with Serial.print() - even though this is a no-op on tinyAVR 0/1-series and megaAVR 0-series parts, it imposes no cost there, and makes it easier to port code to classic AVR and DA-series parts.
  * Don't depend on getting `_flashStringConstant` back from F() 
  * Don't use PROGMEM on tinyAVR 0/1-series and mega 0-series- anything declared const will be kept in flash, yet can be accessed like a normal variable (yay for memory mapping!)
  * if you think your code might end up running on either classic or modern AVRs, use #ifdefs to select the right implementation. (insert example here)
* Use Blink-Without-Delay techniques and millis() for timing.
  * Unless you're doing a very short delay, and/or want it to block until the delay is over. One should never busy-wait for millis() timekeeping - that's just a less precise delay()!
* Use fast digitalRead/Write/pinMode where possible and supported.
* If you fork a library on github...
  * If it's temporary - just for making a PR - delete it after the PR was accepted. PRing bug-fixes is a good-citizen thing to do.
  * If it's for long-term personal use, modify the README.md to say that, and that you don't recommend others use it.
  * If it's for public consumption, modify the README.md to describe the changes you've made and, preferably, why.


## Less-basics
* pulseIn() kind of sucks - particularly on modern AVRs, you can do way better with input capture and a type B timer, but even with a 16-bit timer on the classic AVRs, it's still better than pulseIn(). (see [InputCapture example](https://github.com/SpenceKonde/AVR-Best-Practices/peripherals/InputCapture.md); for one thing input capture is non-blocking if you want it to be (that is, you can do it in the background while also doing something else), as well as being potentially more precise. 

## How to refer to pins
While all of these packages do provide "Arduino Pin Numbers" and "Analog Pin Numbers" (eg, pins named "A0" and such) it is recommended that you refer to pins using their "port pin numbers" PIN_Pxn notation (ie, PIN_PA2 refers to PA2, pin 2 on PORT A). These are immune to confusion relating to different pin mappings (which are common, for historical reasons, and provide maximum portability between different parts in the same families (particularly with the newer parts, where devices with different pin counts from the same family will typically have the same functionality on the same port pins. 

You **cannot** refer to these with "PIN_xn" (ex PIN_B2). 
Some older cores do not support the PIN_Pxn notation; see the recommendations of hardware packages.


