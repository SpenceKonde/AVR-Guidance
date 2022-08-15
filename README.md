# Best Practices, Notes and Guidance
For programming AVRs with Arduino IDE(with some remarks on other IDEs)

## Types of AVR boards
There are three types of AVR-based boards you may end up working with
1. Basic - No bootloader, just the chip, any supporting components to provide extra functionality, and (hopefully) a convenient programming header (see below) to upload code These are often assembled with unprogrammed chips, and then programmed in circuit. Examples include the majority of commercial products, most ATTiny Classic parts, and the vast majority of modern AVRs (notable exceptions being the official Arduino boards).
2. Bootloader based boards - These boards have a bootloader preloaded onto the chip. This enavbles programming through a different interface (usually a Serial port), and is very commonm in Arduino circles. Unless countermeasures have been taken aginst it, they can also be reprogramed through the basic method methioned above (though this may not be as convenient; for example, there may be no dedidated ISP header, forcing you to figure out which pin goes to each pin of the programmer). If the bootloader uses Serial is is likely based on Optiboot (and if not, it ought to be) though others exist, such as ChipBoot. Optiboot using boards usually have a mechanism to reset the chip when the DTR or RTS line is asserted (when the port is opened).
  a. Nano-like boards have a USB connector and serial adapter IC on them. DTR or RTS is tied to reset via a 0.1uF capacitor, which has a 10k external pullup to Vdd.
  b. Pro-mini-like boards have a 1x6 pin FTDI serial header on them for connection to the serial adapter and are generally the eqivalent of that
3. VUSB Bootloader boards - technically a specific type of bootloader, VUSB/Micronucleus/Digispark uses a bitbanged "usb" implementation to upload. It works shockingly well considering how well (or rather how poorly) it conforms tothe USB standard. Despite that much working quite well - indeed well enough that you can ship a product with reset disabled, and still have people reliably upgrade the bootloader by uploading code ocontaining the new bootloader and pieece of code that copies it over the old version. However, it all comes unglued once the sketch starts. I have come to doubt the possibility
4. Native USB boards - These have a microcontroller that interfaces directly to USB - like the classic 8u2, 16u2, 32u4 and so on. Unfortunately, thee is as yet no modern AVR with native USB. There is also a dark side to native USB - if your code hoses the app bad enough, it will only detect as a USB device for 8 seconds after plugging it in, until this is used to upload working code.

## Types of chips and programing connections
1. "Classic" avrs were released before 2016, have a good deal of similarity in how they work, and, as their native programming interface use the SPI pins, except that reset is driven low as if it were the SS pin on a typical SPI device. There are three ways to "soft-brick" these:
  a. Set the clock source to one that is not present. Sice the spec requires that the SCK period while programming be less than 1/3rd of the system clock, if the system clock is 0, ISP programming is impossible.
  b. Set the RSTDSBL fuse, which turns reset into a (really crappy) I/O pin. It has output drive srength of about 1/10th that of a normal pin in this mode.
  c. clear the SPIEN fuse, which disables SPI programming.
  The first case is easy to solve: Simply connect a ciock source that is within spend to the CLKI pin, and burn bootloader with thecorrect clock source selected.  The other two can only be undones with an HV programer. One of these is vaguely, pratical to do with home-made tools.  HVSP, for the 85, 84, and 841. All larger parts reqire HV-PP which can require up to 16 pins and is rarely if ever done in hobby circles.
2. "Modern" AVRs are very reistant to bricking:
  a. Some parts allow reset UPDI to be used as GPIO. If this is done, an HV pulse is needed to reprogram them.
  b. During programming brown-out detect is forced on.
3. Therefore, we do not advise setting PA0 to reset (instead of UPDI) or disabling reset on reset on any classic AVR. Especially one with more than 14 physical pins.

## Sections
### [Glossary of terms](Glossary.md)
Status: Most important/relevant terms, I think, present.
Covers terms used in this and other documents I've written about AVR parts for Arduino, as well as terms common in Arduino
### [Best Practices guide](BestPractices.md)
Status: Mostly incomplete, some sections useful, many others absent or incomplete.
### [Hardware Notes](HardwareNotes/)
Status: Not organized or indexed, and not many items covered yet.
Notes on specific hardware design decisions, one per file.
### [Troubleshooting Help](Troubleshooting/)
Status: Severely incomplete, not useful yet!
Guidance for Arduino troubleshooting
### [Peripheral Guide](Peripherals/)
Status: Doesn't exist yet, some content written but elsewhere currently.
A quick overview of various peripherals and accomplishing common tasks using them.
### [Project Planning](ProjectPlanning/)
Status: Mostly not complete, some thoughts on part selection present.
Choosing microcontroller and core, things like that
