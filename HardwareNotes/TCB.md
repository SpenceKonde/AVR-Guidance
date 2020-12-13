# TCB (Timer/Counter Type B)

## Important differences
There are two major versions of the TCB peripheral. The tinyAVR 0/1-series and megaAVR 0-series have Version 1, the 2-series and Dx-series have Version 2. 
### Version 2 has a second interrupt trigger
Version 1 only had the CAPT bit in INTCTRL/INTFLAGS, which triggered under circumstances depending on the mode:
* On CNT=CCMP in periodic interrupt, single shot, and time-out check modes (note that in the first two cases, it resets to 0 at that point; in the last, it doesn't unless the user manually does so!
* On compare match in 8-bit PWM mode 
* On capture in all othwe modes

Version 2 has CAPT (which works identically, for code compatibility), plus OVF, which is set when the counter reaches MAX and wraps around to 0.
* You can use it to count overflows in input capture modes, allowing timing of longer pulses.
* In periodic interrupt mode, OVF can only fire if the period was changed after the CNT had already passed the new value. At least you know you screwed up and missed a cycle!
* It is used in 32-bit input capture, below.

There is still only the one interrupt vector, TCBn_INT_vect! To make use of the OVF vector, check for it when the interrupt is triggered; often just knowing that it occurred is enough. 

### Version 2 offers new clocking options
* Some parts with Version 2 have two Type A timers (TCA); either of these can be used for a prescaled clock.
* Version 2 can be clocked from an event channel (there is now a second event user for each TCB, for the count). This seemingly small change opens up enormous possibilities, dramatically enhancing the usefulness of the event system and the versatility of the TCB peripheral!

### Cascade mode
These two functions are combined to permit input capture with 32-bit resolution (that is to say, you can time something nearly 3 minutes long to an accuracy of 24ths of a microsecond); it requires 2 TCBs and an EVSYS channel, in addition to the one for the event being captured. Setup is straightforward,  see the datasheet for the procedure.

## Warnings
### TCBn.INTFLAGS
In capture modes, the CAPT flag is cleared when you read the CCMP register. It is not automatically cleared otherwise, and if you don't clear it, the interrupt will fire again as soon as the first one returns. The symptom of this will be that the sketch runs, but grindlgly slowly, because it will always execute one instruction between interrupts... Forewarned is forearmed = if your sketch hangs when you first expect the interrupt to fire, you either forgot to clear the INTFLAGS in your ISR, or you misspelled the ISR vector name (which is a warning not an error; since December 2020, this warning has been forced on, even if you tell the the IDE to not show warnings. You always need to clear OVF (if you are triggering interrupt off it or care if it is set. 
### CAPT INTFLAG not set if CAPT is not enabled as an interrupt source
At least on tinyAVR 0/1-series... https://github.com/SpenceKonde/megaTinyCore/edit/master/megaavr/extras/NotesOnPeripherals.md
