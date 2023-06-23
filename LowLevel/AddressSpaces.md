# Address Spaces in AVR
How many address spaces does an AVR have?

The answer you should all know is Four (five on some parts, sort of, see signature space). Five on modern AVRs. Though only three of them are actually fundamental to the architecture. If you don't know this, you probably should.

## What is an address space?
An address space is simply a chunk of memory, which is accessed with a number, the address. Separate address spaces can use the same number, and have it point to something else, and different addresses in the two address spaces may point to the same underlying byte of memory.

The three address spaces are:

### Program Space (flash, progmem, code space)
The AVR is a harvard architecture, so the program space is a separate address space. This runs from an address of 0x0000 to whatever the size of the flash is. It is nominally a 16-bit address - though as many internal mechanisms use word addressing, the limitations of the architecture don't bite until beyond 128k, for the most part - though it does manifest in that you can't use read_byte_near() for data stored above 64k in the flash, and you need to mess around with farptrs and use read_byte_far() and similar instead.

When self-programming, modern AVRs with fully mapped flash and page buffers write by first filling a page buffer by executing ST instructions - *pointing to the address as it is mapped in the dataspace* - the program space addresses see use only to address jumps, calls, and that sort of thing. Once the page buffer is full, and erase-write page is normally issued, and further pages can be programmed once the write is finished. Writes will fail if an attempt is made to write across page boundaries.

Modern AVRs without page buffers (the Dx-series), can use either ld/st instructions against the mapped portion of flash, and if the FLWR nvm command is selected, writes targeting flash that you are allowed to write to from the current location of program execution (the bootloader cannot overwrite itself, and the app can only write app data - as always) will take place immediately - either a byte at a time with st, or 2 at a time with SPM. Unlike page-buffer using parts, which must load the page buffer and issue the final write command from an appropriately privileged section of flash, on Dx-series, only the SPM or ST instruction itself needs to be issued from that section. You can stage the command in the application section, and then call an address in the bootloader section that is known to contain a SPM; RET; sequence; this is how the DxCore flash library works, using either an entrypoint provided by optiboot, if using that, or a "fake" bootloader section if not (the details are somewhat tricky - but it's handled for you by DxCore). Only erase operations must be done on a whole page at once - though the pages are notably large compared to other AVRs, at 512b/ea.

In all cases, self-programming without erasing will successfully turn 1's into 0's, but cannot change 0's into 1's. This can be taken advantage of to reduce flash wear - it's the erasure process that does the damage that eventually limits the lifetime of the flash, and you only need to erase anything when new-value & old-value != new-value. Hence, you can add to the end of a section of flash one byte at a time, and take advantage of the clean 0xFF's of the erased flash thee, and you only need to suck the whole page into memory, erase it, and rewrite the parts you want to keep in the case where you would need to turn a 0 into a 1 (or to cover most cases very simply, when you need to write to an already-written location, as opposed to one that's never been written). This is much more convenient as well.

For LPM and SPM (on parts that have it) the program space is addressed using the Z pointer and RAMPZ if >64k in size.

### The Data Space
The data space consists of the SRAM, as well as all the special function registers.

Many other things are "mapped" onto the dataspace, particularly in modern AVRs.

On classic AVRs, the 0x20 lowest addresses in the data space are mapped to the 32 working registers. No such facility is provided on modern AVRs, but it doesn't matter - if you are accessing working registers from C in that way, you are probably not doing what you should be doing. Following those is the 0x40 (64) byte "I/O space"), see below.

On modern AVRs, the I/O space is mapped to 0x00-0x40, and the working registers are not mapped at all (because they're not very useful!).

However, the EEPROM and USERROW are mapped to the data space (addresses depend on part family, but they're always in the 0x1000-0x2000 range: All the special function registers except NVMCTRL are located between 0x0000 and 0x1000, and NVMCTRL starts at 0x1000, first with the control and status registers, and then the fuse and lockbytes, the signature row, the user row, and the eeprom are mapped into later points between 0x1000 and 0x2000.

Modern AVRs with up to 48k of flash additionally map the entire program space into the data space, and the linker is smart enough to put variables there if they're declared const! The program space is mapped starting from 0x8000 except on megaAVR 0-series, where to accomodate the 48k parts, it is mapped from 0x4000. This means that the variables do not need pgm_read_x macros, and reads are frequently more performant, as the compiler has less latitude to optimize those.

Modern AVRs with more flash than that, however, can only map 32k of the program space (can be changed at runtime), and do so starting from 0x8000. But because it can be changed at runtime, the compiler can't be sure what section will be mapped, so it can't put things there automatically (DxCore provides ways to do that as outlined in the DxCore PROGMEM docs).

Last - but by no means least - the dataspace contains the SRAM itself (the precice address it starts from varies. On modern AVRs, the highest address in ram tyically just before the first byte of the mapped progmem, so the more ram in a given family a part has, the lower the address that it's ram will start at, while the mapped progmem remains unchanged.

The data space is accessed with the load/store instructions and their variants (lds, sts, ld, ldd, st, std), and for the last four, all pointer registers can be used normally (the X pointer cannot be used for ldd or std - there were not enough opcodes left to do that)

### EEPROM and USERROW spaces
The EEPROM space is generally of small size compared to other physical memories. On classic AVRs, the procedure to read it requires loading EEPROM address registers, and there are a few minor differences between EEPROM memories across the product line, so consult the datasheet for more information.

Nobody really liked that, and on modern AVRs, the EEPROM space is instead only accessible through the memory mapping - it is in a section of the of the 0x1000-0x2000 range used by the NVMCTRL. Access for page-buffer using modern AVRs is almost identical to flash, though the page buffer is half the size, and only bytes that are updated in the page buffer get written.

The USERROW is a smaller section of memory that works similarly, and is only available on the modern AVRs - on the tinyAVRs, it is treated like EEPROM and written in the same way (just with a different library, for complicated reasons) - which is special for two reasons: a UPDI programmer can write (but not read) the USERROW on a locked chip (this is a big deal in production, since you might have your chips pre-programmed and locked, but as you fully assemble units ready for sale, you need to get the serial number or something into it. This is how you'd do that sort of thing). And it is not erased when a CHIPERASE cycle is executed. Only a direct erase command will erase it.

On the Dx-series, the USERROW acts more like flash - erasing is all-or-nothing, and writes to the USERROW are done with FLWR NVM command, and under the hood are implemented with store instructions when accessed from the code. Writes without an erase (which as noted clears the entire USERROW) are only able to turn 1's into 0's. This makes the Dx-series USERROW library a little bit different, as it also has the short 1000 cycle rated endurance that flash does, so facilities are provided by the userrow to make it possible to be more efficient with USERROW writes and automating rewriting of required parts of the USERROW if you try to write over a portion of it.

### The I/O space
Mentioned above in the data space section, this small address space, consisting of just 64 bytes, is very powerful. First, though it can be accessed through it's mapping in the data space, it's faster and smaller to access it directly. IN and OUT can operate on the whole I/O space and both are single-clock instructions. The I/O space is populated by a variety of special function registers. It is mapped to the same addresses in data space on modern AVRs, but on classic AVRs, the addresses run from 0x20 to 0x5F when accessed through the dataspace, but 0x0000 to 0x0040 via IN and OUT.

Always, SP_L, SP_H, (the stack pointer), RAMPZ (if present) and SREG are within the high I/O space (the upper 32 bytes). If the part also has a CCP register, that is also in the high I/O space.
The low I/O space is markedly more exciting: you can directly access bits (though both the address and the bit must be known at compile time for optimization) with SBIS, SBIC, SBI and CBI. The low I/O space generally contains the PORT registers, at least one GPIOR (general purpose 1 byte register which does nothing special beyond living in a place you cand reach it with In and Out and the bit access functions); on classic AVRs the remaining I/O space has commonly used special function registers. On modern AVRs, in contrast, there are 4 GPIORs at the top of the low I/O space, and the rest is dedicated to the VPORT registers, in order to allow single cycle access to the pins - that completely fills the low I/O space with room for 7 ports. Meanwhile the high I/O space is left mostly empty. I'm not sure whether they have any grand/secret plans for it or anything.

### The Signature Space
On many classic AVRs, the same mechanism used to read the fuses within the code can also be directed at other addresses, reading from a poorly documented, and very small signature space. One could argue that the SIGROW on modern AVRs is also it's own data space; but this is a matter of semantics. I don't consider something it's own address space if it is never accessed on it's own terms, can't be written. and all that matters is what range of addresses have infomative read-only values and what don't. The SIGROW is read-only (though there is obviously a way to factory program them - my guess is it's done with a secret UPDI key, and that they can likely lock the door behind them after factory cal).

While accessing it on classic AVRs is painful, it's straightforward and fully documented on the modern AVRs. tinyAVRs have a bunch of mysterious values in there, likely programmed atthe factory cal stage, almost all of which vary only between part families or flash sizes. It is my suspicion that one of the things the cal process involves is telling the chips what kind of part they are and what their capabilities are. However, three bytes do appear to be some sort of secret calibration, and vary within a narrow range. Maybe in addition to the oscillator calibration, they also have to "trim" the oscillator get speeds in the correct range. This could be confirmed by trying to correlate terms of the F<sub>CPU</sub> vs OSCCAL curve (it can be fit perfectly with a polynomial best fit line in excel) to the values of those cal values, though since the cal would presumably bring them closer to the mean, this may be hard. In contrast the Dx-series has mostly an empty wasteland there. If they need to program anything beyond what's documented, it is not mapped to the data space and is hidden from our prying eyes.

### The BOOTROW
This is a new section coming to some of the most recently announced AVR devices. The DU-series will have  BOOTROW, as will the recently announced AVR EB-series. It's unclear what the restrictions and capabilities of this section of memory will be.


## A few relevant details
### Dx-series (and future Ex-series parts with >32k flash) parts can access only part of their flash through the dataspace. This defaults to the highest 32k section after startup, and is best left there. The truncated addresses that you get when you take the address of a variable declared in PROGMEM_SECTION1 (64k parts) or PROGMEM_SECTION3 (128k ones) can be used without modification. The other sections require the addresses to be offset by 0x8000. This has a strange consequence; the memory looks like this on a 128k Dx-series:
```
Program Space:
 _______________________________
|     0x0000-0x7FFF             | <-- BOOTCODE section always starts at 0x0000
| Declare PROGMEM, access w/LPM | <-- BOOTSIZE marks the end of the bootloader section, which is typically only a single page. Only code executing from here can write the APPCODE section, but unless reads from
|_______________________________|     APP are prohibited by the bootloader setting the BOOTRP bit, only 2 instructions are needed to create a callable entry point here and use a fake bootloader section in the first 512b
|     0x8000-0xFFFF             |     allowing the app to rewrite itself freely (not that it's wise to do so),
| Declare PROGMEM, access w/LPM |
|_______________________________|
|     0x10000-0x17FFF           |
| Cursed ground. Only farptrs   |
| and ELPM/read_x_far() work    |
|_______________________________|
|     0x18000-0x1FFFF           |
| Declare PROGMEM_SECTION3      | <-- CODESIZE can be set with 512b granularuty, marking where APPCODE gives way to APPDATA. It must be between BOOTSIZE and the highest address of the flash.
| Access as a normal variable   |
|_______________________________| <-- APPDATA section always ends at 0x1FFFF (on 128k modern AVRs). So without using SPM from app = ALL, you can't make anything outside this section writable without this whole section being writable.

A variable stuffed into PROGMEM_SECTION3, if one uses the & opperator on it, will yield:
&foo = 0x8000 (assuming it's at the start of the section)

However get_farptr(foo) will instead give you 0x18000.

Of couse, looking at the dataspace, we see that the first number is exactly what we want if we wish to access it. By far the most natural way to access the flash is to put those constants into section 3, and access them normally, and if you need more space for program memory, put it into PROGMEM normally, and access with LPM. For an application with close to 96k of data in flash, and 32k of code, using thatstrategy, the code mostly ends up in the cursed ground, which is fine because you don't need to read the executable code normally, while there are straightforward ways to access the memory.

One thing worth noting also though is that you get, in effect, two markers which can be placed with page granularity. The first one marks the end of the bootloader section, from whence the application section can be written, and the start of the application section, which cannot write to itself, but can write to EEPROM, USERROW, and the third section of flash. The second marker marks the end of the application code section, and the beginning of the application data section. The application can write to this section, and you can even execute code from it. But the code running here cannot write to any NVM, not even the EEPROM.

Nothing can write to the bootloader section except via UPDI. Ever (this is required by the security model they use now - if you could write to the bootloader on a locked chip somehow, you could replace it with a bootloader that would dump the flash for you, which would be unacceptable for commercial users.
 _______________________________
| 0x00 - 0x40: I/O space        |
|_______________________________|
|     0x0000-0x0FFF             | <-- Notice that the SFRs overlap with the I/O space. The things in the I/O space are also SFRs - they're registers, they have a special function, they're SFRs.
| Extended I/O registers/SFRs   |
|_______________________________|
|     0x1000-0x1FFF             | <-- The layout in here varies by part family.
| NVMCTRL, EEPROM, and USERROW  |
| as well as SIGROW mapped here |
|_______________________________|
| 0x2000-0x3FFF - the abyss     | <-- This gap appears to contain no data and is not writable.
|_______________________________|
|     0x4000-0x7FFF             | <-- 16k of SRAM. On a 64k part with 8k of SRAM, this instead starts at 0x6000, and the abyss is larger.
| SRAM - all variables go here  |
|_______________________________|
|     0x8000-0xFFFF             |
| PROGMEM_SECTION3 mapped here  |
|                               |
|                               |
|_______________________________|

```
