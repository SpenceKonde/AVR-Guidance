# A comparison of modern AVRs 
Separate from classic AVRs because they are not directly comparable, and because they blow the doors off any classic AVR that challenges them.

## Argh, I broke the table. I don';t know what I broke though, nor how to find it so I can correct it
Sorry :-(

| Series                  |     0-Series   |     1-Series    | 1-Series "plus"  |      2-Series       |     AVR__DA/DB   |   AVR__DD20/14 |  AVR__DD28/32  |    AVR__EA             | AVR__EB     |       megaAVR 0   |
|-----------|-------------|----------------|----------------|-----------------|------------------|---------------------|------------------|----------------|------------------------|-------------|-------------------|
| Availability            | Many backorder |  Many Backorder | Many Backordered |    Many Backordered | Many backordered | Many Backorder | Many Backorder |          Unreleased  | Unreleased  |  Some backordered |
| Pincounts               | 8*  14, 20, 24 |  8*  14, 20, 24 |       14, 20, 24 |          14, 20, 24 |  28, 32, 48, 64  |       14 or 20 |       28 or 32 |      28, 32, or 48   |14, 20, 28 or 32 | 32, 40, or 48 |
| Flash                   | 2*   4, 8, 16k |       2*  4, 8k |       16k or 32k | 4k, 8k, 16k, or 32k |   32k, 64k, 128k |  16k, 32k, 64k |  16k, 32k, 64k |   8k, 16k, 32k, 64k    | 8k, 16k, 32k | 8k, 16k, 32k, 48k |
| RAM                     | 128,256,512,1k |    128,256,512b |            2048b | 512, 1k, 2k, or 3k  |      4k, 8k, 16k |   2k,  4k,  8k |   2k,  4k,  8k |   1k,  2k,  4k   8k     | 1k, 2k, 3k  |  1k,  2k,  4k, 6k |
| Separate reset pin?     |             NO |              NO |               NO | optional on 20/24pin| reset/GPin   PF6 | reset/GPin PF6 | reset/GPin PF6 |    reset / GPin PF6 |reset / GPin PF6|  reset / GPin PF6 |
| Use UPDI as GPIO?       | Yes, but reset | Yes, but reset  | Yes, but reset   |    Yes, but reset   |               No |     if enabled |     if enabled |          if enabled     | if enabled  |                No |
| PWM pins (Arduino) *    | 8pin: 4 else 6 |      4, 6, or 8 | 14 pin: 6 else 8 |                   6 |            10-18 |          5 - 8 |         8 - 10 |              9 - 14      | 4, 6, 8 or 10 TBD `@` |                8 | 
| Type A timers           |       1 x v1.0 |        1 x v1.0 |         1 x v1.0 |            1 x v1.1 |       1-2 x v1.1 |       1 x v1.1 |       1 x v1.1 |            2 x v1.1  | No, TCE w/WEX |       1 x v1.0 |
| Type B timers           |       1 x v1.0 |        1 x v1.0 |         1 x v1.0 |            2 x v1.1 |       3-5 x v1.1 |       2 x v1.1 |       3 x v1.1 |            4 x v1.1  | 2 x v1.1      |        3/4 x v1.0 |
| Type D timer            |             No |             Yes |              Yes |                  No |       Yes, w/PLL |     Yes, w/PLL |     Yes, w/PLL |                  No   | No, 1 TCF     |                No |
| Real Time Clock         |    † Yes, v0.8 |     † Yes, v0.8 |     † Yes, v0.9  | ‡ Yes, v0.9 or v1.0 |         Yes v1.0 |       Yes v1.0 |       Yes v1.0 |           Yes, v1.1 | Yes, v1.1|    |       Yes v1.0 |
| USARTs    (pin options) |          1 (2) |           1 (2) |            1 (2) |               2 (3) |   3-6, 1 or 2/ea |       2 (`††`) |        2 (2, 4)|  2/3 (5, 2 or 3, 2)  | 1 (2-6) |   3/4 (1 or 2/ea) |
| SPI pin mapping options |              1 | 2 except 14-pin |   2 except 14pin |   1 (2 on 20/24pin) |            ‡‡‡ 6 | 3 (4 on 20pin) |              5 | ?    1, 2 on 48-pin |             4 |                 3 |
| TWI pin mapping options |              1 |  2 except 8-pin |                2 |                   1 | 1/2, up to 3 mux/ea | ! 3 (4 on 20p) |       !!  4 | !! except 48 pin  4 | 1, up to 3 mux |                2/3 |
| Maximum rated speed     |         20 MHz |            20Mz |           20 MHz |              20 MHz |           24 MHz |         24 MHz |         24 MHz |              20 MHz    |       20 MHz |            20 MHz |
| Overclocking (internal @ 5v) |   ???     |       25-30 MHz |        25-30 MHz |              32 MHz |           32 MHz |   Maybe 32 MHz |   Maybe 32 MHz |  TBD, likely 32 MHz |  Likely 25-30 MHz |  Likely 25-30 MHz |
| Overclocking (ext. clk @ 5v) |   ???     |          32 MHz |           32 MHz |           >= 32 MHz |           48 MHz |   Maybe 48 MHz |   Maybe 48 MHz |  TBD, likely 32 MHz | TBD Likely 32 MHz |     Likely 32 MHz |
| External crystal        |       RTC Only |       RTC Only  |         RTC Only |            RTC Only | RTC, + HF on DB  | RTC or HF only |  RTC and/or HF |       RTC and/or HF |      RTC Only |          RTC only |
| Event Channels          |1 sync, 2 async | 2 sync, 4 async |   2 sync 4 async |       6 normal ones | 8/10 normal      |       6 normal |       6 normal |       6 normal v1.1 | 6 normal v1.1|      6 normal v0.99 |
| CCL Logic Blocks        |     2 (1 pair) |      2 (1 pair) |       2 (1 pair) |         4 (2 pairs) | 4/6 (2/3 pairs)  |    4 (2 pairs) |    4 (2 pairs) |         4 (2 pairs) |       4 (2 pairs) |       4 (2 pairs) |
| Analog Comparators      |   1, no DACREF |     1, w/DACREF |      3, w/DACREF |         1, w/DACREF |     3, w/DACREFs |    1, w/DACREF |    1, w/DACREF |         2, w/DACREF  | 2, DACREF likely |        1 w/DACREF |
| ADC                     |     10-bit ADC |      10-bit ADC |   2x 10-bit ADCs |   12-bit diff w/PGA |   ‡‡ 12-bit diff | ‡‡ 12-bit diff | ‡‡ 12-bit diff |   12-bit diff w/PGA |   12-bit diff w/PGA |        10-bit ADC | 
| DAC (analog output)     |             No |  Yes, 8-bit *** |   Yes, 8-bit *** |                  No |       Yes 10 bit |     Yes 10 bit |     Yes 10 bit |                 Yes |              No |             No |
| Max ADC clock speed     |        1.5 MHz |         1.5 MHz |          1.5 MHz |             3/6 MHz |           2 MHz  |          2 MHz |          2 MHz |        6 or 3/6 MHz | Likely 6 or 3/6 MHz |           1.5 MHz |
| Analog References       |          Set A |           Set A | Set A + external |              Set B2 |           Set B  |          Set B |          Set B |              Set B2 | Set B2           |  Set A + External |
| Multivoltage features   |           None |            None |             None |                None | DB:4/8 MVIO,INLVL|  3 MVIO, INLVL |  4 MVIO, INLVL |               INLVL | No MVIO. INLVL likely |              None |

`*` - these 8-pin parts aren't available with more than 4k of flash. The 2k parts are not available in pincounts higher than 14.

`**` - All 8 pin parts give 4 (a future update may enable an alternate configuraton on 1-series 8-pin parts to yield 3 buffered TCA pins and 2 TCD pins). All 14 pins have 6 PWM pins,  and the TCD0 pins are on the same port as some of the TCA ones. Future versions of the core will improve flexibility through a tools submenu.

`***` - The tinyAVR DAC can use only internal references, not Vdd or external, and is the same as the DACREF.

`@` - Until details on the TCE and TCF are available, we might have as few as 4, or even 3 PWM channels, but up to 8 or 10 more likely. 

`!` - On 14 pin DD's, there is one slave mode only pair, one master only pair, and 1 that does both. On the 20-pin ones, an additional master-only mapping is added and one with both master and slave is replaces the slave only one.

`!!` - On 28 and 32 pin parts, of the 4 mappings, only 2 support dual mode.

`†` - These parts are afflicted by significant errata which must be understood before using the RTC.

`††` - The 14-pin parts have 3 mappings for USART0, (1 with only TX/RX. aanother with no XDIR), and 2 for USART1 (the first has no TX pin, the latter only TX and RX). The 20-pin ones hve 5 for USART0 and 2 for USART1. 

`‡` - The 16k and smaller versions still don't have the RTC fix, making them much trickier to use. , 3216/3217 do.

`‡‡` - I refer to this as a pseudodifferential ADC. It's capabilities are as if two ADCs sampled simultaneously and the result was subtracted. You have a differential ADC, but gain is unity, and and Vref must exceed the voltages measured. Whereas the 2-series and EA have a "real" differential ADC, which can work with inputs a fraction of a volt beyond the power rails, regardless of VREF, and typically have a gain stage, allowing small voltage differences to be measured even right near the power rails (including things like the other side of a current sense resistor, which might be 0.1 or 0.2v above Vdd.  

`‡‡‡` - 2 SPI ports each with up to three pinsets. **these are treated like one SPI peripheral by SPI.h (in DxCore)** because all the libraries you'd want to use it with will get confused because SPI1 can't be the name of the second SPIclass instance because it's already used for the SPI_t struture (basically every library that supporteed more than one SPI port being on the chip barfed failed to compile when told there were two interfaces.


## What The Overclocking Numbers Mean
Overclocked speeds are typical maximum speeds that can be reached with basic functionality intact. Not all parts will reach or function at these speeds, and the operating temperature range is likely far narrower than it is at the rated speeds. I'm aware of nobody who has played with overclocking the 0-Series, but as it was released at the same time as the 1-Series and appears to be a 1-Series with fewer features, I would expect them to be the same - though if they turned out not to be, I could rationalize a difference in either direction equally easily (and even phrase it in a way that makes someone who assumed otherwise look silly: "they're budget parts, made with a less advanced process, of course they don't overclock as well!" or "without all those wacky features hanging off the bus, of course they overclock better!").

The internal oscillator speeds can be reached (on most but not all specimens) with [tuning](megaavr/extras/Ref_Tuning.md). Note that while modern tinyAVRs can be tuned readily, this is not viable for overclocking on Dx - but the internal speed selection has two undocumented speeds, 28 and 32 MHz. Best results overclocking are (as on classic AVRs) obtained with an external oscillator. Never use an overclocked part in any critical application; neither stability nor correctness of arithmetic can be guaranteed! We still don't know whether the EA's tuning will be tiny-like or DD-like (initial indications are tiny-like) - though it could be something else entirely.

## Voltage Reference Options
| VREF sets   | Lowest * |  Low   |  Mid   | High | Highest | External | Used in part families       |
|-------------|----------|--------|--------|------|---------|----------|-----------------------------|
| Set A       | 0.55V    |   1.1V | 1.5V   | 2.5V |    4.3V | not all  | tinyAVR 0/1, megaAVR 0      |
| Set B       | -        | 1.024V | 2.048V | 2.5V |  4.096V |     Yes  | AVR Dx-series               |
| Set B2 **   | -        | 1.024V | 2.048V | 2.5V |  4.096V |     Yes  | tinyAVR 2, AVR Ex           |
| Classic 1   | -        |   1.1V | -      | -    |      -  |      No  | Many low end classic parts  |
| Classic 2   | -        |   1.1V | -      | -    |  2.56V  |     Yes  | Many better classic parts   |

`*` - the 0.55V reference voltage must be used with a much lower ADC clock
`**` - the internal voltage reference on the 2-series halves the maximum ADC clock speed, thoughwith an original maximum of 6 MHz, it remains twice as fast as the far less capable parts 0/1-series ADC

## 1-Series Parts With 16k or 32k of Flash
Unlike almost every other AVR ever, there are additional "bonus" features based on the flash-size of parts within this family: The 16k and 32k versions (only) have a few extra features: they all have 2k of ram, whether 16k or 32k. They have 3 analog comparators (including a window mode), a second - desperately needed - type B timer. But weirdest of all they have a second ADC exactly like the first, differing only in which pins the channels correspond to! Was this an answer to those who wanted a differential ADC? (trigger both from same event, subtract, and get a differential reading with the same limitations as the Dx-series's "differential" mode only with fewer bits). You can also have one in freerunning mode (maybe using the window comparator, or both freerunning, continually measuring different pins. With separate sample and hold capacitors, crosstalk is minimal, while it is very signficant using a single ADC to alternately measure two pins. 

## Pincount options
Series      | 8 | 14 | 20 | 24 | 28 | 32 | 40 | 48 | 64 | 100 | Largest flash |
------------|---|----|----|----|----|----|----|----|----|-----|---------------|
tinyAVR 0/1 | X |  X |  X |  X |    |    |    |    |    |     |       16k/32k |
tinyAVR 2   | X |  X |  X |  X |    |    |    |    |    |     |           32k |
megaAVR 0   |   |    |    |    |    |  X |  X |  X |    |     |           48k |
AVR DA/DB   |   |    |    |    |    |  X |    |    |  X |     |          128k |
AVR DD      |   |  X |  X |    |  X |  X |    |    |  X |     |           64k |
AVR EA      |   |    |    |    |  X |  X |    |  X |    |     |           64k |
AVR EB      |   |  X |  X |    |  X |  X |    |    |    |     |           32k | 
Classic Tiny| X |  X |  X |    |  X |  X |    |    |    |     |           16k |
Classic Mega|   |    |    |    |  X |  X |    |  X |  X |   X |          256k |

Note that 100 pincount parts, having typically 80+ pins, will only provide single cycle bit-level write and read for - at most - 7 ports. on modern AVRs, Additional ports after PORTG would have to be "second class pins" and can only be accessed through the PORT register, which is slower (though bitset and bit clear can be atomic, unlike the second clas pin on classic AVRs.
Note also that 128k flash is the most flash that can be supported without a 3-byte program counter. This results in call and ret operations executing more slowly, 

## Spreadsheet of all modern AVRs with memory specs
This spreadsheet is gathered from public data, and lists memory specs and device ID's. Gaps in device ID's are noted, with speculation about the reasoning.
[https://docs.google.com/spreadsheets/d/14tkhND4I4KUwLesKOpxHj2eC5IMZf9aOpDS6wgclc5A/edit?usp=sharing](https://docs.google.com/spreadsheets/d/14tkhND4I4KUwLesKOpxHj2eC5IMZf9aOpDS6wgclc5A/edit?usp=sharing)

## Peripheral versions
**These versions are 100% unofficial - it's a numbering system I personally started assigning because there are subtle differences that are very easily missed while looking through the datasheets (the difference may be a single line in a 700 page document**
USART: I'm not even going to try to number these. Errata have appeared and disappeared from several erratum listings without a die rev, so not even Microchip seems to know what it impacts. 
TCA 1.1 supports an additional event input.
Both TCA 1.0 and 1.1 have an "erratum": The restart command/event behaves as described in the datasheet. That is wrong (apparently) and it should not reset the direction
TCA 1.1A (or 1.0A, if the TCA is fixed without adding the new event input) fixes that erratum. Presumably it will also be fixed in the datasheet as well
TCB 1.1 adds cacscade mode, clock on event, and the overflow flag.
TCB 1.1A finally fixes the registers in 8-bit PWM mode to act as 8-bit registers. 
EVSYS 0.9 has two kinds of channels, and two kinds of users, and a dumb layout of generator options for the channels. EVSYS 0.99 and 1.0 are identical except that the 
RTC 0.8 does not support a crystal, but is afflicted by a brutal erratum. This is easily worked around **but only if you are aware of it**. I've gone back and forth with someone on this for days before we concluded that they couldn't do what they were planning with the RTC, because it was sunk by one part of the erratum, but depended on another part of the erratum. 
RTC 0.9 supports a crystal, but is still afflicted by the erratum
RTC 1.0 is identical to RTC 0.9 except that the erratum is corrected. (yeah, the erratum is that bad - it has at least two major aspects and is maddening to debug.
RTC 1.1, EVSYS 1.1, GPIO 1.1 - these come as a group. On EVSYS 1.0, each event channel gets 16 options corresponding to pins on each port, and half of the PIT-divided-by-n options. In 1.1, each channel has 2 event generators for each channel, and 2 event generators for the RTC. The RTC and PORT peripherals get a register to configure the event generation. In other words, starting with the Ex-series, the event system is finally what it should have been from the beginning.  

## A complaint: Too f'ing many documents, not enough centralized information
The modern AVRs are have VERY few differences. megaTinyCore and DxCore share a large amount of code, and where the code is different, it is frequently a matter of optimization rather than anything else. In spite of that:
* There is no document to my knowledge that includes even the most basic specifications for all of the tinyAVR parts within even just the 0/1-series parts. 
* Parametric search on digikey gives only the coarsest-grained information, and of course, nothing specific to AVR, which is where all the interesting stuff would be. 
* The device ID patterns hold interesting insights (though I can imagine that Microchip might not want to talk so much about *those*.)
* Despite the fact that a large portion of the (rather numerous) errata effects multiple parts (often "All AVRs released from 2016 to 2022"), there is no master list on Microchip's website that has an erratum on each row, and a column for each (presumed) die (a die is presumed to be the same on two parts if they share errata sheets and datasheets and exist in the same silicon revisions), which would show one mark for "All versions known to be impacted" another for "All versions known NOT impacted", and, if it's been fixed, the first fixed REVID, and for newly added errata where the testing hasn't been conducted on all parts yet, a mark for "Belived to effect"/"belived not to effect". I know of people who held off on buying AVR-DD parts until someone proved to them that an erratum was fixed there. It is often months between errata revisions, so you can see an erratum pop up on one part.... and then you have to test whatever part you're actually using to see if it has the bug too or not - the fact that it's not on the errata for that part could mean it's uneffected, and they tested it, that they haven't tested it but are pretty sure it is (or isn't) present there, or that they have confirmed it and it will be in the version of the errata they'll publish in a few months.Really? 
  * They remove "datasheet clarifications" as they are addded to the datasheet. This can lead to the awful situation where you've seen two versions of the datasheet, but there were two errata sheets released in that time, the first of which added clarifications, and the second which came out with the datasheet, and removed the clarifications. The Datasheet Revision History provides neither a thorough description of the changes nor a link to the specific place in the datasheet that was changed. What are you supposed to do? You can't even compare the text without writing a program to desecure and rip the contents of the PDFs to text and then perform extensive regex replacement, and even then might nor be viable.  
  * I know Microchip would love to not talk about the errata or datasheet clarifications (Guess which direction the clarifications are typically in?), but with the modern AVR era containing very few die revs that have actually fixed problems, of which many parts had 10-20, several serious, they really owe it to their customers to provide a better way of tracking it. Even better would be if they marked what priority the errata were within Microchip, so we could plan ahead and use that to inform our design decisions (for example, if a part has a showstopper errata for you, you want to know "Do I design around a different part?" (maybe a newer AVR)
