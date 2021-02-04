# Modern AVR Peripheral versions
As time goes by, it has become clear that, unlike classic AVRs, where peripherals for a given chip were generally... sort of ad-hoc, here there is a clear progression of versions. The information contained in this document is entirely empirical. I am using semantic versioning, because it seems appropriate and useful.

| Family  | ADC     | DAC | CCL/EVSYS | CLKCTRL | NVMCTRL | PORTMUX | TCA  | TCB | TCD  | VREF |
|---------|---------|-----|-----------|---------|---------|---------|------|-----|------|------|
| Tiny0/1 | 1.0     | 1.0 | 0.9       | 1.0     | 1.0     | 1.0     | 1.0  | 1.0 | 1.0  | 1.0  |
| mega0   | 1.0     | NA  | 1.0       | 1.0     | 1.0     | 2.0     | 1.0  | 1.0 | NA   | 1.1  |
| AVR DA  | 1.1     | 1.1 | 1.1       | 1.1     | 2.0     | 2.0     | 1.0  | 1.1 | 1.1  | 2.0  |
| AVR DB  | 1.2     | 1.1 | 1.1       | 1.2     | 2.0     | 2.0     | 1.0  | 1.1 | 1.1  | 2.0  |
| AVR DD  | 1.3     | 1.1 | 1.1?      | 1.2     | 2.0?    | 2.1?    | 1.0? | 1.1 | 1.1? | 2.0  |
| tiny 2  | 2.0     | NA  | 1.1       | 1.0     | 1.0     | 2.0t    | 1.0  | 1.1 | NA   | 2.0  |
| AVR EA  | Not 2.0 | ??? | 1.1?      | ???     | ???     | ???     |      |     |      |      |

## ADC

## DAC
1.0 is 8-bit, and doesn't let you use VCC as the reference.
1.1 is 10-bit, and lets you use VCC as the reference. Much more convenient!

## CCL/EVSYS
The tinyAVR 0/1-series had a what appears to be a prototype of the event system - with those separate sync and async channels. And a different (and generally worse) arrangement of inputs for the CCLs. And no interrupts on the CCLs...
All subsequent parts have rectified these glaring deficiencies.

## PORTMUX
1.0 - uses CTRLA/etc and sometimes has more than one peripheral per register. Like CCL/EVSYS, looks like a prototype. Apparently mega
2.0 - uses (peripheral)ROUTEA/etc names, one kind of peripheral per register
2.1 - AVR DD, has more options for pin mappings for SPI, USART


## TCA
There don't seem to have been any changes to TCA

## TCB
1.1 brings two changes with big ramifications: The ability to count on events, which was conspicuously absent from 1.0, and the ability to use that to cascade two timers to get 32-bit input capture. It also brings one with smaller ramifications: An OVF interrupt flag (still fires the same interrupt, though) - I haven't tested it, but this may let you set it to TOP-(time desired) and poll the OVF flag to time an interval without using an interrupt (which doesn't work in 1.0 unless interrupts are globally disabled, and you enable the interrupt)

## TCD
There don't seem to have been any changes to TCD, though availability of clock sources depends CLKCTRL

## VREF
VREF 1.x uses 0.55, 1.1, 2.5, 4.3, and 1.5V.
Those voltages sucked.
VREF 2.x uses 1.024, 2.048, 4.096 and 2.500V. The powers of two are exceptionally useful when working with analog sensor readings.
