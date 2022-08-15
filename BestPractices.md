# Best Practices for AVRs in Arduino

## THIS GUIDE IS IN VERY EARLY STAGES
## It should NOT be used as a general guide at this time
**Some pages may be ready for public consumption, and linked to from elsewhere, but as a general guide, it is not**

## Motivation
Poor programming practices are endemic to the Arduino community, and result in poor sketch performance, wasted flash/RAM, and difficulty porting sketches to other parts. These can also make your code difficult for others to understand, for example if you post on the Arduino forums requesting assitance. Poor practices common in Arduino can also increase the liklihood of bugs that result in the MCU crashing or resetting, as well. For best results, read this guide **BEFORE** you write the sketch (this is not intended as a debugging guide, but rather a set of practices to reduce the need for debugging).

It is recommended that all design be performed targeting *chips* not *boards* (unless - maybe - you're going to be getting that board manufactured as be selling them) - this eases the process of moving from a general purpose development board (like an Arduino board) to a custom produced board for your application - unless you're sure that your design will be a one-off, you don't want to have to the specific quirks of the development board you happen to have.

This guide applies to all AVR (including "megaavr" which is a made-up arduino name for a subset of their products that they have staunchly refused to give a name to) parts supported by Arduino via official board packages or third party hardware packages. (if you are the author of a third party hardware package that supports parts not listed, and want to be included on this list, please contact me). This does not cover the XMEGA processors - and to my knowledge they are not supported by any Arduino library.

## Basics

### Memory

* `F()` and `__flashStringHelper` are not quite standard - or I should say - their absence isn't standard: When the hardware has fully memory mapped flash (modern AVRs with 48k of flash or less), any `const` variable (including string literals) will be left in flash, and not copied to RAM. Contrived examples can easily be created that compile on a 16k tinyAVR with 1k of ram, but fail to compile for a 64k flash Dx-series with 8k of RAM, complaining that there's not enough RAM! (just call Serial.println("some text") with a very long string) - on DxCore you will see increasingly large differences in the used flash (and generated code) as you switch between a 32k part (which is fully memory mapped) and a 64k part (which isn't fully mapped).
  * Use F() macro for string constants printed with Serial.print() even when you don't need it.
  * ~Don't depend on getting `_flashStringHelper` back from F()~ (forget it, we were forced to remove this optimization by a library author).
  * Don't use PROGMEM on tinyAVR 0/1/2-series and mega 0-series- anything declared const will be kept in flash, yet can be accessed like a normal variable (yay for memory mapping!)
  * if you think your code might end up running on either classic or modern AVRs, use #ifdefs to select the right implementation. (insert example here)
* Use Blink-Without-Delay techniques and millis() for timing.
  * Unless you're doing a very short delay, and/or want it to block until the delay is over. A short delay() isn't *always* bad. Often, yes, but not always.
  * One should never busy-wait for millis() - that's just a less precise delay(), just use delay() at that point.
    * exception: You're using some weird combination of core + options with millis() and delay(), but not micros(), particularly if you have ISR's you want to be triggered during that delay.
    * On cores without micros(), delay() calls delayMicroseconds(1000) repeatedly.

### libraries on github
* **If you fork a library on github**...
  * If it's temporary - just for making a PR - *delete it after the PR is accepted, so nobody mistakes your fork for the version they should be using.*
  * If it's for long-term personal use, but not meant for others to use, *modify the README.md to say that, and refer them to the upstream library.*
  * If it's for public consumption, *modify the README.md to describe the changes you've made and why.*
* **When you use a library you found on github**
  * Make sure it's one being actively maintained.
  * If the readme hasn't changed from the upstream version, that's not a good sign. In fact, if a readme hasn't been changed since it was forked, you probably should be using the upstream version unless you've been told explicitly to go use it.
  * There are often dozens or hundreds of forks of popular libraries, most of them not meant for public consumption or maintained. These are likely to have undocumented changes and/or be out of date. People routinely ask about problems with a library, I search for it, and find dozens of hits all on github... and when I interrogate them about the version they're using, they tell me yet another one.


### Less-basics
* pulseIn() kind of sucks; it causes millis counts to be missed if the pulse is longer than 1-2ms, it's blocking and the resolution is only 1us on modern AVRs, you can do way better with input capture and a type B timer, but even with a 16-bit timer on the classic AVRs, it's still better than pulseIn(). (see [InputCapture example](https://github.com/SpenceKonde/AVR-Best-Practices/blob/master/peripherals/InputCapture.md); for one thing input capture is non-blocking if you want it to be (that is, you can do it in the background while also doing something else, and even if you don't want to do the next step of your program you probably want the millis timer to keep ticking), as well as being potentially more precise. On many cores, there is a bug in the code that causes the timeout pass significantly faster before the start of the pulse.

### How to refer to pins
While all of these packages do provide "Arduino Pin Numbers" and "Analog Pin Numbers" (eg, pins named "A0" and such) it is recommended that you refer to pins using their "port pin numbers" in PIN_Pxn notation if the core you are using supports it (ie, PIN_PA2 refers to PA2, pin 2 on PORT A). These are immune to confusion relating to different pin mappings (which are common for historical reasons) and provide maximum portability between different parts in the same family: Since the 2016 revolution, alternate functions have not been getting removed...

You **cannot** refer to these with "PIN_xn" (ex PIN_B2).
Some older cores do not support the PIN_Pxn notation; see the recommendations of hardware packages.
