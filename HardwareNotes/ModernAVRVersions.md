# Modern AVR Peripheral versions
As time goes by, it has become clear that, unlike classic AVRs, where peripherals for a given chip were generally... sort of ad-hoc, here there is a clear progression of versions. Parts rarely "regress" - notice how the extra pin mappings the DD-series gained to make the small pincount versions viable are in the full size EA-series too. The information contained in this document is entirely empirical. I am using semantic versioning, because it seems appropriate and useful.

| Family  | ADC     | DAC | CCL/EVSYS | CLKCTRL | NVMCTRL | PORTMUX | TCA  | TCB | TCD  | VREF | RTC |  AC |
|---------|---------|-----|-----------|---------|---------|---------|------|-----|------|------|-----|-----|
| Tiny0/1 | 1.0     | 1.0 | 0.9       | 1.0     | 1.0     | 1.0t    | 1.0  | 1.0 | 1.0  | 1.0  | 0.9 | 1.0 |
| mega0   | 1.0     | NA  | 1.0       | 1.0     | 1.0     | 2.0     | 1.0  | 1.0 | NA   | 1.1  | ??? | 1.1 |
| AVR DA  | 1.1     | 1.1 | 1.1       | 2.0     | 2.0     | 2.0     | 1.1  | 1.1 | 1.1  | 2.0  | 1.0 | 1.2 |
| AVR DB  | 1.2     | 1.1 | 1.1       | 2.1     | 2.0     | 2.0     | 1.1  | 1.1 | 1.1  | 2.0  | 1.0 | 1.2 |
| AVR DD  | 1.3     | 1.1 | 1.1?      | 2.1     | 2.0     | 2.1?    | 1.1? | 1.1 | 1.1? | 2.0  | 1.0 | 1.2 |
| tiny 2  | 2.0     | NA  | 1.1       | 1.1     | 1.0     | 2.0t    | 1.1  | 1.1 | NA   | 2.1  | 1.0 | 1.2 |
| AVR EA  | 2.x     | 1.1 | 1.1?      | ???     | ???     | 2.1?    | ???  | 1.1?| NA   | 2.1? | 1.0 | 1.2?|

## ADC
1.0 is 10 bit, up to 1.5 MHz
1.1 is 12 bit,
1.2 adds
1.3 adds more ADC channels - they wrap around from PORTF to PORTA pin 2
2.0 is a total redesign with true differential mode and a programmable gain amplifier, enhanced accumulator mode, and ability so read individual samples while accumulating.

## DAC
1.0 is 8-bit, and doesn't let you use VCC as the reference, and is shared with an AC's DACREF
1.1 is 10-bit, and lets you use VCC as the reference.

## CCL/EVSYS
0.9 feels like a prototype -  with those separate sync and async channels. And a different (and generally worse) arrangement of inputs for the CCLs. And no interrupts on the CCLs...
1.0 fixes these issues and makes a much better peripheral. D-latch still busted.
1.1 D-latch works.
1.2 Ind
All subsequent parts have rectified these glaring deficiencies.

## PORTMUX
1.0 - uses CTRLA/etc and sometimes has more than one peripheral per register. Like CCL/EVSYS, looks like a prototype. Apparently mega
2.0 - uses (peripheral)ROUTEA/etc names, one kind of peripheral per register
2.1 - AVR DD, has more options for pin mappings for SPI0, USART0

## CLKCTRL
1.0 - no crystal. internal 16/20 MHz oscillator or ext. clock. 64-level calibration for internal oscillator. 20 MHz at maximum cal runs 30-32, and usually crashes at 31+ @ 5V and room temp
1.1 - no crystal. internal 16/20 MHz oscillator or ext. clock. 128-level calibration for internal oscillator. 20 MHz at maximum would run at 38-40, but it crashes in mid 30's @ 5V and room temp.
2.0 - no crystal, but can autotune internal oscillator withe external watch crystal. Unfortunately, only 32 levels of tuning, and they're too far apart to get really accurate internal clock this way. The internal oscillator can be set to 1, 2, 3, or 4 MHz, or any multiple of 4 above that to 32 (though only up to 24 is documentedd or supported officially). Comes with a PLL rated for 48 MHz and 2x or 3x multiplication. PLL has hidden 4x multiplication option too. PLL output can only be used for TCD0, nothing else (pity you can't clock the CCL with it; the CCL hardware will oscillate at ~ 100 MHz if you set it to invert it's output, so I think it could keep up).Runs well at 32 MHz internal; PLL clocking timer D works for basic functionality up to at least 128 MHz (32 x 4).
2.1 - Adds support for a crystal, and clock failure detection. On some October-2020 vintage DB64's, 40 MHz with external clock appears viable. 3x PLL multiplier works, has glitches particularly when changing dutycycle. 48 MHz does not.

2.x may come alongside the Dx-style core

AVR EA talks about 16/20 MHz ,

Note that in all versions, if you set the 32KHz oscillator to use external clock, you can run it up to CLK_MAIN/4 (and you can jumper a PWM output to the external clock pin to supply that if you want). Mainly notable if you meed to count system clock cycles to very high numbers.

## TCA
1.1 adds a second event input.

## TCB
1.1 brings two changes with big ramifications: The ability to count on events, which was conspicuously absent from 1.0, and the ability to use that to cascade two timers to get 32-bit input capture. It also brings one with smaller ramifications: An OVF interrupt flag (still fires the same interrupt, though) - I haven't tested it, but this may let you set it to TOP-(time desired) and poll the OVF flag to time an interval without using an interrupt (which doesn't work in 1.0 unless interrupts are globally disabled, and you enable the interrupt) like you could on clasic parts.

## TCD
There don't seem to have been any changes to TCD, though availability of clock sources depends CLKCTRL,

## VREF
VREF 1.x uses 0.55, 1.1, 2.5, 4.3, and 1.5V.
Those voltages sucked.
VREF 2.x uses 1.024, 2.048, 4.096 and 2.500V. The powers of two are exceptionally useful when working with analog sensor readings.
