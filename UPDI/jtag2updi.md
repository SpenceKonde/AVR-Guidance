# Step-by-step guide to turn a Uno/Nano/Pro Mini into a UPDI programmer

The tinyAVR 0/1/2-series, megaAVR 0-series, and AVR Dx-series parts are programmed through the Unified Program and Debug Interface (UPDI). This is a 1-wire interface using the UPDI pin on the AVR Dx-series part. A UPDI programmer is required to change the fuses ("burn bootloader"), upload a bootloader (if desired) and upload sketches if a bootloader is not in use. The classic ISP programmers cannot be used – only UPDI can be used to program these parts.

## Two ways to make a UPDI adapter
There are two easy ways to get UPDI programming for just a few dollars.

### From a serial adapter
As of DxCore 1.3.0 and megaTinyCore 2.2.6, it is now possible to use a serial adapter and 4.7k resistor.

Connections:
* VCC, GND of serial adapter to VCC (or VDD), GND of AVR target.
* 4.7k resistor between TX and RX of adapter (many adapters have built-in 1k, 1.5k, or 2.2k resistor in series with TX; these should use a proportionally smaller resistor)
* RX of adapter to UPDI pin of AVR target. A small resistor (under 1k; 470Ω is generally recommended) in series with this is fine.

Choose "Serial Port and 4.7k" from the Tools -> Programmer menu, and select the Serial Port from the Tools -> Port menu.

**Note** This does not provide a serial monitor. Connect a serial adapter (or two) the normal way to monitor via serial, along with an external serial terminal application. This technique works with inexpensive CH340 serial adapters from eBay, AliExpress, etc. This is a good way to utilize serial adapters that don't have a DTR pin broken out.

### From a Nano or Pro Mini
An Arduino sketch is available to turn ATmega328(P)-based Arduinos, such as the Arduino UNO and Nano, into an UPDI programmer. However, it does not work on boards based on other parts, such as the 32u4 (Micro/Leo) or any non-AVR board. The following steps show how to make one of these low cost UPDI programmers. It is recommended to use an Arduino Nano or Pro Mini (a cheap clone from eBay is fine) and to hard-wire it for the task.

## Part 1: Upload the sketch to your Arduino
1.	Download and extract the [UPDI programmer sketch](https://github.com/SpenceKonde/jtag2updi), or clone the repo to the local machine.
2.	Browse to the download location and open the jtag2updi folder.
3.	Open the sketch jtag2updi.ino and upload it to the Arduino. The .ino file itself is empty, and this is correct – all the code is contained in other files in the same folder, but the empty .ino is needed so that the IDE can compile it.

## Part 2: Connect hardware
*Previous versions of this guide specified a cap between reset and ground after programming. Testing has revealed this to be unnecessary.*
1.  Connect GND of Arduino to GND of the AVR target.
2.  Connect Pin 6 (D6) of the Arduino to the UPDI pin of the AVR target - if using the bare chip, connect it via a [470Ω resistor](https://github.com/SpenceKonde/AVR-Best-Practices/blob/master/HardwareNotes/UPDISeriesResistors.md). Many breakout boards will provide a separate UPDI pin that has this resistor built-in; in this case, this pin may be connected directly to the programming pin.
3.	Unless the ATtiny has its own power supply, connect the 5V pin of the Arduino to the VCC (or VDD) pin of the AVR target.

Now, select the appropriate board (such as ATtiny megaAVR series) from Tools -> Board, and upload a sketch via the IDE. The same programmer can also be used to Burn Bootloader; be sure to select the jtag2updi (megaTinyCore) programmer from the Tools -> Programmer menu.

**If the process appears to hang at the start of the upload, press and release the reset button on the UPDI programmer** This can still sometimes happen, despite extensive bug fixes to jtag2updi.

![Minimal UPDI connections](NanoUPDI_Minimal.png "Minimal UPDI connections – no resistors")

![Reccomended UPDI connections](NanoUPDI_Recommended.png "Recommended UPDI connections – 470Ω in series with UPDI")

### Ignore the warning about "flash" and "boot" memories
A warning will be shown during the upload process `avrdude: jtagmkII_initialize(): Cannot locate "flash" and "boot" memories in description` – this warning is spurious and can be safely ignored.

## Permanent programmer assembly suggestions
* For convenience, it is recommended to dedicate a Nano or Pro Mini to this purpose, and to solder the connections. Nano and Pro Mini clones can be had on eBay for a few dollars, shipped. Use one without the headers pre-installed, or solder it to a perfboard.
* Solder the wires in place – 0.1" DuPont jumpers are recommended: Cut the jumpers in half, strip, and solder in place.
* After soldering the wires in place, glue them to the bottom of the board with hot-melt glue, otherwise they will fatigue and break easily with handling.
* It is highly recommended to arrange the connectors in the following order: UPDI, GND, VCC – this prevents damage if the jumper is accidentally connected backwards. Use a 3-pin DuPont housing, or hold three 1-pin housings in together with adhesive tape.

## Bootloaders and "Burn Bootloader"
"Burn Bootloader" does three things:
1. It sets the fuses to match the options selected in the Tools submenus (options controlled by fuses are marked in the menu).
2. Erases the flash.
3. Writes the bootloader to the flash, if a board definition with a bootloader was selected – e.g. the ones marked with (Optiboot) in the board menu.

In order to stop using the bootloader after having installed it and go back to uploading via UPDI, choose the "no bootloader" board option, and "burn bootloader" to set the fuses appropriately. If in doubt, there is never harm in burning the bootloader, because unlike classic AVRs, the board cannot be bricked by "burning bootloader" with improper settings.

## Troubleshooting

### Ignore the warning about "flash" and "boot" memories
A warning will be shown during the upload process whenever uploading via jtag2updi: `avrdude: jtagmkII_initialize(): Cannot locate "flash" and "boot" memories in description` – this warning is spurious (a bug in avrdude, or in the modifications made to support jtag2updi). It should be ignored.

### Use Verbose uploads
The avrdude output is very terse by default, particularly with jtag2updi. Enable verbose upload in order to get useful feedback during the upload (and bootloading) process.

## Photographs

### Nano as UPDI programmer, assembly and use
![Nano as UPDI](NanoAsUPDI.png "Nano converted to UPDI programmer")

### Pro Mini (and serial adapter) as UPDI
![Pro Mini as UPDI](ProMiniAsUPDI.png "Pro Mini converted to UPDI programmer")

### Typical development configuration
Because it is frequently useful to have a serial port for debugging, it is common to have a serial adapter and UPDI programmer connected simultaneously. Example:

![Development configuration for tinyAVR 0/1-series](https://github.com/SpenceKonde/megaTinyCore/blob/master/megaavr/extras/DevConfigUPDI.png "A common development configuration for tinyAVR 0/1-series")

Or, it is possible to use Optiboot, but without disabling UPDI to get reset. Finally, other tricks are possible such as using alternate reset.
