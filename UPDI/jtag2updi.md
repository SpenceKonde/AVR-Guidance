# Step-by-step guide to turn serial adapter or uno/nano/pro mini into a UPDI programmer

The tinyAVR 0/1/2-series, megaAVR 0-series, and AVR Dx-series parts are programmed through the Unified Program and Debug Interface (UPDI). This is a 1-wire interface using the UPDI pin on the AVR Dx-series part. A UPDI programmer is required to change the fuses ("burn bootloader"), upload a bootloader (if desired) and upload sketches if a bootloader is not in use. The classic ISP programmers cannot be used - only UPDI can be used to program these parts. Luckily it is much easier to make a UPDI programmer than an ISP programmer!

There are two very easy ways to get UPDI programming hardware for $3 or less.
## From a serial adapter (recommended)
As of DxCore 1.3.0 and megaTinyCore 2.2.6, it is now possible to use a serial adapter and a schottky diode! (or a 4.7k resistor - but the diode works better)

As of megaTinyCore 2.3.2 and DxCore 1.3.6, these are much faster than jtag2updi, and are the recommended method of programming.
### Serial adapter requirements
Almost any cheaper-than-dirt serial adapter can be used for pyupdi style programmer, as long as you take care to avoid these pitfalls:
1. The FTDI FT232, (both the genuine ones, and the fakes) are by default configured to use less CPU time, but this absolutely destroys performance. You can easily fix this (at least on windows - I'm not sure if the problem even happens on Linux): Open device manager, under Ports (COM and LPT), locate the FTDI adapter. Right click -> properties. Click the Port Settings tab, and then the Advanced button. Middle of left hand side, set "Latency timer" to 1ms). Click OK enough times to leave the dialog. you'll see the adapter disappear and reappear as the change is applied. Configured properly, the FT232RL has spectacular performance. Configured improperly, the performance is downright abysmal.
2. Many serial adapters have a resistor, typically between 1k and 2.2k in series with their TX line; If yours has one, just reduce the value of the resistor between Tx and Tx by about that much if you go the resistor-only method. If you use a diode instead which you should, an adapter with a resistor like this is preferred: in that case, you only need to connect the diode, as the builtin resistor will replace any external one.r. 
3. Some serial adapters have a dedicated LED to indicate receiving. While some fancy chips have an I/O pin that drives the RX led (the FT232 has that feature I think), a cheap adapter with an RX LED may have just put an LED and resistor on the RX line. The load from an LED on the UPDI line will overwhelm any signal and prevent communication  (a LED on TX wired like that is fine as long as it is connected to Tx before the series resistor  - which is a design flaw I've not seen in circulating poorly designed serial adapters; and I've seen a lot of badly designed serial adapters.)

### Connections:
* Vcc, Gnd of serial adapter to Vcc, Gnd of target
* 4.7k resistor between Tx and Rx of adapter (many adapters have built-in 1k, 1.5k, or 2.2k resistor in series with Tx; these should use a proportionally smaller resistor)
  * For better results, a smaller resistor (that built-in one on most adapters, mentioned above, will do perfectly here) and a small schottky diode (band towards Tx, other end connected to Rx) can be used (use a "small signal diode" - larger general purpose diodes may have properties that make them less suitable for this) The diode substantially widens the tolerances of this programming method, and significantly improves reliability. 
   * My top pick here is the BAT54C,235; it's in as tiny SOT-23 package (it's 2 diodes both weith the "band" towards the pin thats' alone on one side) Why? Because, assuming your serial adapter has the pins on 0.1" header, and TX and RX are next to eachother (both extremely common) the the diode fits right in beteeen them. and with no lead that could later fatigue and break the result is less likely to be damaged by rough handling. Then if I want to more ovbviously maek it as a UPDI programmer, I might cut off the Tx, DTR and CTS pin
* Rx of adapter to UPDI pin of target. A small resistor (under 1k - like the 470 ohm one we generally recommend) in series with this is fine.

Choose "Serial Port and resistor or diode" from the Tools -> Programmer menu, and select the Serial Port from the Tools -> Port menu.

Note that this does not give you serial monitor - you need to connect a serial adapter the normal way for that (I suggest using two, along with an external serial terminal application). This technique works with those $1 CH340 serial adapters from ebay, aliexpress, etc. Did you accidentally buy some that didn't have a DTR pin broken out, and so weren't very useful with the Pro Minis you hoped to use them with?
Connections:


## From a nano or pro mini
An Arduino sketch is available to turn ATmega328(p)-based Arduino’s, like the Arduino UNO and Nano, into an UPDI programmer (it does not work on boards based on other parts, like the 32u4 (Micro/Leo) or any non-AVR board). The following steps show how to make one of these low cost UPDI programmers. We recommend using an Arduino Nano or Pro Mini (a cheap clone from ebay is fine) and hard-wiring it for the task.

### Part 1: Upload the sketch to your Arduino
1.	The UPDI programmer sketch can be found here: https://github.com/SpenceKonde/jtag2updi
Download and extract, or clone the repo to your local machine.
2.	Browse to the download location and open the jtag2updi folder
3.	Open the sketch jtag2updi.ino and upload it to your Arduino. The .ino file itself is empty, and this is fine - all the code is contained in other files in the same folder, but the empty .ino is needed so that the IDE can compile it.

### Part 2: Connect hardware
*previous versions of this guide specified a cap between reset and ground after programming. Testing has revealed this to be unnecessary*
1.  Connect Ground of Arduino to Ground of the ATtiny
2.  Connect Pin 6 of the Arduino to the UPDI pin of the ATtiny - if using the bare chip, connect it via a [470 ohm resistor](https://github.com/SpenceKonde/AVR-Best-Practices/blob/master/HardwareNotes/UPDISeriesResistors.md). Many breakout boards will provide a separate UPDI pin that has this resistor built-in; in this case, this pin may be connected directly to the programming pin.
3.	Unless the ATtiny has it's own power supply, connect 5v pin of the Arduino to the Vcc pin of the ATtiny

Now, you should be able to select an ATtiny megaAVR series board from Tools -> Board, and upload a sketch via the IDE. The same programmer can also be used to Burn Bootloader (be sure to select the jtag2updi (megaTinyCore) programmer from Tools -> Programmer menu)

**If the process appears to hang at the start of the upload, press and release the reset button on the UPDI programmer** I pounded on jtag2updi for like a month trying to get rid of all the bugs like this, and after finally getting my fixes merged in, discovered that somehow, this could still happen.

![Minimal UPDI connections](NanoUPDI_Minimal.png "Minimal UPDI connections - no resistors")


![Recommended UPDI connections](NanoUPDI_Recommended.png "Recommended UPDI connections - 470 Ohm in series with UPDI")

#### Ignore the warning about "flash" and "boot" memories
A warning will be shown during the upload process `avrdude: jtagmkII_initialize(): Cannot locate "flash" and "boot" memories in description` - this warning is spurious and can be safely ignored.

### Permanent programmer assembly suggestions:
* For convenience, we recommend dedicating serial adapter, Nano or Pro Mini to this purpose, and soldering the connections. Nano and Pro Mini clones can be had on ebay for $2-5 shipped. Use one without the headers pre-installed. Serial adapters are $1-2 shipped. 
* Solder the wires in place - we suggest using 0.1" DuPont jumpers, because you'll want a dupont connector on the other end, and crimping your own dupont connectors is profoundly unpleasant: Cut the jumpers in half, strip, and solder in place.
* After soldering the wires in place, glue them to the bottom of the board with hot-melt glue, otherwise they will fatigue and break easily with handling.
* We suggest arranging the connectors in the following order: UPDI, GND, Vcc - this way, if you attach the connector backwards, no harm is done. Use a 3-pin DuPont housing, or hold three 1-pin housings in together with scotch tape.

## Bootloaders and "Burn Bootloader"
"Burn Bootloader does three things - it sets the fuses to match the options selected in the Tools submenus (options controlled by fuses are marked in the menu), erases the flash, and writes the bootloader to the flash, if you have selected a board definition with a bootloader (the ones marked with (Optiboot) in the board menu). In 2.3.x megaTinyCore and 1.3.x DxCore, some fuses are also set during normal uploads. 

If you wish to stop using the bootloader after having installed it and go back to uploading via UPDI, as long as you are using  megaTinyCore  2.3.0+ or DxCore 1.3.0+ you can just upload with a non-bootloader board selected and it will clear the bootloader automatically - there is no longer the requirement that you "burn bootloader" unless you have changed other settings marked as requiring a burn bootloader to set. For other cores, you must "Burn Bootloader" with the non-bootloader board selected.

## Troubleshooting

### Ignore the warning about "flash" and "boot" memories
A warning will be shown during the upload process `avrdude: jtagmkII_initialize(): Cannot locate "flash" and "boot" memories in description` - this warning is spurious (a bug in avrdude, or in the modifications made to support jtag2updi). It can and should be ignored - it is shown whenever uploading via jtag2updi

### Use Verbose uploads for jtag2updi
The avrdude output is very terse by default, particularly with jtag2updi. Enabling verbose upload ensures that you get useful feedback during the upload (and bootloading process) 

### Verbose mode on serial adapter as UPDI
The verbose mode was far too verbose generating logs several times larger than the program being uploaded, and the normal mode is about as talkative as AVRdude in verbose mode; In DxCore 1.3.6 and megaTinyCore 2.3.2, there is now no difference bwetween normal and verbose output, and you can leave verbose output on all the time. 

## Photographs

### Nano as UPDI programmer, assembly and use
![Nano as UPDI](NanoAsUPDI.png) "Nano converted to UPDI programmer")

### Pro Mini (and serial adapter) as UPDI
(ProMiniAsUPDI.png["Pro Mini converted to UPDI programmer")
### Typical development configuration
Since it is frequently useful to have a serial port for debugging, I typically find myself using a configuration like this, with a serial adapter and UPDI programmer connected simultaneously. Obviously, one could also use Optiboot, but without disabling UPDI to get reset, or using other awkward tricks (see [AlternativeReset. So
![Development configuration for tinyAVR 0/1-series](DevConfigUPDI.png "A common development configuration for tinyAVR 0/1-series")
