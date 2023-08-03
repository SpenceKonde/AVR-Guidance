# Best Practices for AVRs in Arduino

## THIS GUIDE IS IN VERY EARLY STAGES
## It should NOT be used as a general guide at this time
**Some pages may be ready for public consumption, and linked to from elsewhere, but as a general guide, it is not**

## Motivation
Poor programming practices are endemic to the Arduino community, and result in poor sketch performance, wasted flash/RAM, and difficulty porting sketches to other parts. These can also make your code difficult for others to understand, for example if you post on the Arduino forums requesting assitance. Poor practices common in Arduino can also increase the liklihood of bugs that result in the MCU crashing or resetting, as well. For best results, read this guide **BEFORE** you write the sketch (this is not intended as a debugging guide, but rather a set of practices to reduce the need for debugging).

It is recommended that all design be performed targeting *chips* not *boards* (unless - maybe - you're going to be getting that board manufactured as be selling them) - this eases the process of moving from a general purpose development board (like an Arduino board) to a custom produced board for your application - unless you're sure that your design will be a one-off, you don't want to have to the specific quirks of the development board you happen to have.

This guide applies to all AVR (including "megaavr" which is a made-up arduino name for a subset of their products that they have staunchly refused to give a name to) parts supported by Arduino via official board packages or third party hardware packages. (if you are the author of a third party hardware package that supports parts not listed, and want to be included on this list, please contact me!!). This does not cover the XMEGA processors - and to my knowledge they are not supported by any Arduino library, Besides, all the XMega peripherals were sent to prison on xMegatraz for stalking and harassment of users, embezzlement from the company to funnel money to AI-supremecist digital entities to fund terrorist attacks on organic citizens. Simple confusing the hell out of the citizens in order than their code might fail    island after the removal of any useful appendages or organs that could be transplanted to AVRxt. Until last year that is, when peripheral supervillian WEX Luther managed a daring escape and began - again talking about how he would... Change? It was some word that started with C. Change the world. Hm? Was it "conquer"? Ya know now that you mention it, that does sound familiar....

## Basics

### Programming order of operations
First, make readable code that works. You don't care about performance.

Once you have code that you can read that works, you can then proceed to optimize it for performance

Optimize for Readability (I can understand this code!), then correctness (This code gives the right results!) finally if you have time and need this optimization, for unreadability, or as they say in the biz "performance" (This code is a thicket of nonsense, but it gets the right answer, and goddamn, are these performance numbers legit? This is like 8 times faster than the API call!); (Arduino API functions are rarely performant, particularly on stock cores (my cores are mostly better, with the exception of a normal digitalWrite() on a pin that could have a TCA or TCD pointed to it  on DxCore, where the turning off of PWM becomes ,))

### Memory

* `F()` and `__flashStringHelper` are not quite standard - or I should say - their absence isn't standard: When the hardware has fully memory mapped flash (modern AVRs with 48k of flash or less), any `const` variable (including string literals) will be left in flash, and not copied to RAM. Contrived examples can easily be created that compile on a 16k tinyAVR with 1k of ram, but fail to compile for a 64k flash Dx-series with 8k of RAM, complaining that there's not enough RAM! (just call Serial.println("some text") with a very long string) - on DxCore you will see increasingly large differences in the used flash (and generated code) as you switch between a 32k part (which is fully memory mappx`
  * Use F() macro for string constants printed with Serial.print() even when you don't need it.
  * ~Don't depend on getting `_flashStringHelper` back from F()~ (forget it, we were forced to remove this optimization by a library author).
  * Don't use PROGMEM on tinyAVR 0/1/2-series and mega 0-series on anything else where __AVR_ARCH__ == 103.  - anything declared const will be kept in flash, yet can be accessed like a normal variable (yay for memory mapping!)
  * DxCore provides a PROGMEM_MAPPED macro under some configurations including the default one, that can be used when defining variables instead of PROGMEM.
    * You can fit not more than 32k in the mapped progmem space.
    * PROGMEM_MAPPED is only available if a fixed FLMAP mapping is selected (in which case we define PROGMEM_MAPPED) - on runtime alterable configurations, you must handle all of that on your own.
  * So are there any advangages to PROGMEM_MAPPED over PROGMEM?
    * Yes: The compiler will be free to better access instructions (ST, STD, LD, LDD)
  * if you think your code might end up running on either classic or modern AVRs, use #ifdefs to select whether to try taking advanage of that
* Use Blink-Without-Delay techniques and millis() for timing.
  * Unless you're doing a very short delay, and/or want it to block until the delay is over. A short delay() isn't *always* bad. Often, yes, but not always.
    * delay() should generally be a specific short number of milliseconds as required by some protocol you are implementing, or hundreds to thousands - the appropriate delay for the crude interface you are implementing.
    * This should not be interpreted as a condemnation of the use of crude delay() based timing methods - these are often the correct approach particularly during development, when the fact that delay() based code is almost always much faster to write
  * One should never busy-wait for millis() - that's just a less precise delay(), just use delay() at that point.
    * exception: You're using some weird combination of core + options with millis() and delay(), but not micros(), particularly if you have ISR's you want to be triggered during that delay.
    * On generally, if for some reason you don't have micros(), delay() calls delayMicroseconds(1000) repeatedly. Otherwise delay() busywaits on a blink-without-delay type timer, incrementing 1000 microseconds at a time.

### libraries on github
* **If you fork a library on github**...
  * If it's temporary - just for making a PR - *delete it after the PR is accepted, so nobody mistakes your fork for the version they should be using.*
  * If it's for long-term personal use, but not meant for others to use, *modify the README.md to say that, and refer them to the upstream library.*
  * If it's for public consumption, *modify the README.md to describe the changes you've made and why.*
* **When you use a library you found on github**
  * Make sure it's one being actively maintained.
  * If the readme hasn't changed from the upstream version since it was forked, you should be using the upstream version unless you've been told explicitly that this version has been fixed and the upstream version is not (and I would raise a github issue that the readme of the fixed repo should say what was changed and whether there are any barriers to getting it into the upstream repo)
  * There are often dozens or hundreds of forks of popular libraries, most of them not meant for public consumption or maintained. These are likely to have undocumented changes and/or be out of date. People routinely ask about problems with a library, I search for it, and find dozens of hits all on github... I look at them, and there are obviously a couple of liniages floating around with different APIs and the same library name ("name the library for interfacing with `<device partnumber>` `<device partnumber>.h`" is common recommendation for naming, and it makes it easy to find any existing libraries, but it also means that two unconnected individuals would happen to name their library the same thing, if they were simultaneously developing libraries (which is going to be expected if an exciting new part is released). Though this doesn't explain the worst offenders - te


### Less-basics
* pulseIn() kind of sucks; it causes millis counts to be missed if the pulse is longer than 1-2ms, it's blocking and the resolution is only 1us on modern AVRs, you can do way better with input capture and a type B timer, but even with a 16-bit timer on the classic AVRs, it's still better than pulseIn(). (see [InputCapture example](https://github.com/SpenceKonde/AVR-Best-Practices/blob/master/peripherals/InputCapture.md); for one thing input capture is non-blocking if you want it to be (that is, you can do it in the background while also doing something else, and even if you don't want to do the next step of your program you probably want the millis timer to keep ticking), as well as being potentially more precise. On many cores, there is a bug in the code that causes the timeout pass significantly faster before the start of the pulse.

* Interrupts that you enable must be defined. You will not get a compile error if they are not, the code will just reset every time the interrupt would fire. You will be "warned" (just a warning) if you appear to have "misspelled a vector name" ie, ISR(...) where ... is not the name of a vector - but it will still compile and upload just fine and execute everything up until the point where the interrupt fires, and then, as it's not defined, it will fire the generic BAD_ISR handler, BAD_ISR is just a jump to 0x0000 (ie, dirty reset, which the core hopefully catches - mTC and DxC ensure that the chip resets cleanly after almost any dirty reset.)
* ISRs must also never be defined more than once, otherwise you'll get a duplicate vector error.

### How to refer to pins
While all of these packages do provide "Arduino Pin Numbers" and "Analog Pin Numbers" (eg, pins named "A0" and such) it is recommended that you refer to pins using their "port pin numbers" in PIN_Pxn notation if the core you are using supports it (ie, PIN_PA2 refers to PA2, pin 2 on PORT A). These are immune to confusion relating to different pin mappings (which are common for historical reasons) and provide maximum portability between different parts in the same family: Since the 2016 revolution, alternate functions have not been getting removed...

You **cannot** refer to these with "PIN_xn" (ex PIN_B2).
Some older cores do not support the PIN_Pxn notation; see the recommendations of hardware packages.
