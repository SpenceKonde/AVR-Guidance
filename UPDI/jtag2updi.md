# Step-by-step guide to turn serial adapter or uno/nano/pro mini into a UPDI programmer

The  tinyAVR 0/1/2-series, megaAVR 0-series, and AVR Dx-series parts are
programmed through the Unified Program and Debug Interface (UPDI). This is a
1-wire interface using the UPDI pin on the AVR Dx-series part. A UPDI
programmer is required to change the fuses ("burn bootloader"), upload a
bootloader (if desired) and upload sketches if a bootloader is not in use. The
classic ISP programmers cannot be used - only UPDI can be used to program
these parts. Luckily it is much easier to make a UPDI programmer than an ISP
programmer!

There are two very easy ways to get UPDI programming hardware for $3 or less.



## From a serial adapter (recommended)
As of DxCore 1.3.0 and megaTinyCore 2.2.6, it is now possible to use a serial adapter and a schottky diode! (or a 4.7k resistor - but the diode works better)
As of megaTinyCore 2.3.2 and DxCore 1.3.6, these are much faster than jtag2updi, and are the recommended method of programming.

### Required components
1 USB serial adapter These can be had for as low as $1 on ebay and aliexpress based on the CH340G, slightly more for CP2102
1 fast signal schottky diode such as a 1N4148 or any of many others (recommended) or a resistor (see below to figure out value)).
1 resistor, a few hundred ohms - 220 to 1k, even 2k is fine (may not be needed). A few jumper wires.

### Serial adapter requirements
Almost any cheaper-than-dirt serial adapter can be used for pyupdi style programmer, as long as you take care to avoid these pitfalls:
1. The FTDI FT232, (both the genuine ones, and the fakes) are by default configured to use less CPU time, but this absolutely destroys performance. You can easily fix this (at least on windows - I'm not sure if the problem even happens on Linux): Open device manager, under Ports (COM and LPT), locate the FTDI adapter. Right click -> properties. Click the Port Settings tab, and then the Advanced button. Middle of left hand side, set "Latency timer" to 1ms). Click OK enough times to leave the dialog. you'll see the adapter disappear and reappear as the change is applied. Configured properly, the FT232RL has spectacular performance. Configured improperly, the performance is downright abysmal.
2. MOST SERIAL ADAPTERS ALREADY HAVE THEIR OWN RESISTOR in series with Tx! Typically between 1k and 2.2k; If using the recommended diode method, you need only add the diode, not the second resistor. If you aren't going with the diode method for some reason, use a resistor such that the total is 4.7k (you'll need to measure it with multimeter unless you can follow traces to the resistor and read out the code (the three number codees - first two numbers are the most significant digits, and the third is the number of zeros they are followed by - 222 is 2200 - 2.2k, 471 is 470, (which of course means that 101 means a 100 ohm resistor, and 100 means a 10 ohm one!)).  There are three approaches to measuring all wuith one end of multimeter clipped to tx. Either look up the pinout of the serial adapter chip online and just measure resistance to that pin, check all the pins on the chip. Many may have very high resistance, but only one will have a value of a few k or less, or you go for the ends of the resistors.
(if you actually encounter one without a resistor between Tx and the chip, let me know! I only know of one design circulating in the wild: it's the green PCB with a 3.3v/5v switch and microUSB port; that board was clearly designed by someone with little experience, it has several other major flaws)
3. Some serial adapters have a dedicated LED to indicate Rx. While some fancy chips have an I/O pin that drives the RX led (the FT232 has that feature I think), a cheap adapter with an RX LED may have just put an LED and resistor on the RX line (in fact that's what that above mentioned green boards with switch and microusb port did).. The load from an LED on the UPDI line will overwhelm any signal and prevent communication  (a LED on TX wired like that is fine as long as it is connected to Tx before the series resistor, which is a wierd enough mistake that I have not seen it on any of the many (bad) designs I've examined.)

I recommend the bog standard CH340G as the go-to serial adapter for several reasons:
* They are dirt cheap and readily available on ebay/aliexpress/amazon. 7/$5 shipped on aliexpress for the most basic design (the black kind with the stupiud voltage jumper on the end. The wider ones with rounded corners and ENIG surface treatment are no better electrically, though the build quality has to be better (the first kind you should plan to solder the USB connector onto more solidly), and are bulkier. The better kind are what you find on ebay when you search for "CH340 6pin" - you want the black ones with the voltage switch. Until, that is, I bring my own design into production - it'll have voltage switching done *right* and a micro USB connector instead of a stuipid full sized USB-A, full-sized 1117 3.3v regulator, and a color-coded power light to indicate the current voltage setting.
* The bare chips are dirt cheap and readily available - and also dead simple to design with. Anyone, even the clown who designed those green boards I've been badmouthing, can do it (it looked like someone's first board, frankly). Heck, a working CH340G serial adapter was the first board I designed, too (at least I didn't try to sell my first version!).
* They are - at most - a half second slower to write a full flash image with. For the price of one real FT232RL adapter, you could have half a dozen CH340's.

Most of the FT232RL boards for sale are fake. I have several on order to compare with my real one. If they turn out to perform comparably, I will update this (though even the fakes are expensive compared top the CH340s)

The HT42B534 is a bag of extremes. It doesn't support arbitrary baud rates. Before the optimizations to serial-updi, it absolutely crushed all of the competition. But as I improved performance of the other parts, they could catch up, while the Holtek chip had little room between it's head and the physics-imposed performance ceiling. I am very surprised that the write performance on tinyAVR wasn't better. At some point I will retest - but this all isn't why I don't recommend these parts. There are two issues:
1. The modem control inputs are backwards! At power on, they all show as off. Then, the moment one of them changes state the first time, they all show inverted logic - when an external device asserts (grounds) one, it shows as off, otherwise it shows as on. Opposite of how every other manufacturers' adapter behaves.
2. They have far and away the highest incidence of driver hangs that required the board to be replugged. If you try too high of a baud rate. If you try too low of a baud rate. It you look at them crosseyed. If you look at them without crossing your eyes. If they were actually still faster, I could overlook this (I will overlook a lot to get faster uploads)... but on the Dx-series, they don't support 345600 baud, so the best they can do is 460800, but with --blocksize 32 to slow it down so it doesn't out-write the NVM controller. Whereas on the tinyAVR parts, above 115200 baud, they error out (my theory is that they try to send the next command while the chip is still writing the flash page - on the Dx-series, it programs a word at a time (so it is sustained write speed that breaks - bit s get dropped when the controller reaches 24 bit-periods (1 data word) ahead of the target's NVMCTRL), whereas on the tinyAVR, it is the latency betweeen sending the write page command, and when it starts trying to write again.


### A note on breakout boards
Some tinyAVR and other UPDI-based part breakout boards have an on-board resistor. Sometimes this is a 4.7k one. That is NOT appropriate. I was part of the problem for a while. I think the original mistake came from people conflating the pyupdi resistor with a generally appropriate one. When I started megaTinyCore, my early collaborator was making hardware with a 4.7k resistor; I assumed he was doing it right. While this does work with dedicated programmers, including jtag2updi, it doesn't work with serial UPDIO. It will work with dedicated programmers like jtag2updi, as long as they don't have their own resistor. Suffice to say, for a time it was a very common belief. I use 470 ohms now, but I can't find fault with a design over it not having one at all.

### Connections:
* Vcc, Gnd of serial adapter to Vcc, Gnd of target
* 4.7k resistor between Tx and Rx of adapter (many adapters have built-in 1k, 1.5k, or 2.2k resistor in series with Tx; these should use a proportionally smaller resistor)
  * For better results, a smaller resistor (that built-in one on most adapters, mentioned above, will do perfectly here) and a small schottky diode (band towards Tx, other end connected to Rx) can be used (use a "small signal diode" - larger general purpose diodes may have properties that make them less suitable for this) The diode substantially widens the tolerances of this programming method, and significantly improves reliability.
   * My top pick here is the BAT54C,235; it's in as tiny SOT-23 package (it's 2 diodes both weith the "band" towards the pin thats' alone on one side) Why? Because, assuming your serial adapter has the pins on 0.1" header, and TX and RX are next to eachother (both extremely common) the the diode fits right in beteeen them. and with no lead that could later fatigue and break the result is less likely to be damaged by rough handling. Then if I want to more ovbviously mark it as a UPDI programmer, I might cut off the Tx, DTR and CTS pin; always remember that you can get serial adapters for a buck a piece on ebay.
* Rx of adapter to UPDI pin of target. A small resistor (under 1k - like the 470 ohm one we generally recommend) in series with this is fine.


```
USB Serial Adapter
With internal 1-2k resistor on TX
This is the case in 90% of USB serial adapters.



This is the ideal case - the resistor is alreadt built into serial adaopter (less work) and it will just work with most configurtatioms. while still being safe.

--------------------                                 To Target device. If you make a 3-pin connector, use this pinout
                DTR|                                 ------------     so that it accidentally reversing it is safe.
    internal    Rx |--------------,------------------| UPDI           Switch Gnd and UPDI, if  connected backwards, you would apply reverse polarity power and burn it out
  Tx---/\/\/\---Tx |-------|<|---'          .--------| Gnd            Switch Gnd and Vcc, if connected backwards, UPDI would be hard-grounded while ground was connected
    resistor    Vcc|---------------------------------| Vcc            to positive voltage of idle serial line, hence, it would be below ground, e
  typ 1-2k      CTS|                     .`          ------------     With this scheme, if backwards, UPDI (idle high) would be connected to Vcc, and Vcc to UPDI. Either:
                Gnd|--------------------'                                 * Device is externally powered at similar voltage (if not similar, you'd probably be in trouble
--------------------                                                          no matter what). Target UPDI and adapter RX both held high, that's their idle state, no worries.
                                                                          * Device was to be powered by programmer, and is not a tinyAVR - in this case, the leakage current
                                                                              from RX does little to pouwert the part, and there is some current injection the UPDI pin
                                                                              but probably not very much, and modern AVRs are designed to withstand 15-20mA of that!
                                                                          * As above, but it's a tinyAVR = Reset pin does not have normal protection diodes because of the
                                                                              HV programming thing. No current flows, nothing happens.

                                                  Never design a cable pinout such that it can physically be plugged in backwards but will not survive that electronicallty


Also works great, convenient if still using jtag2updi without resistor built into it. Resistorss should sum to less than 4.7k, preferalby much less

--------------------                                   To Target device.
                DTR|                                 ------------         __________
    internal    Rx |--------------,------------------| UPDI----\/\/\/----|UPDI      |
  Tx---/\/\/\---Tx |-------|<|---'          .--------| Gnd       <2k     |          |
    resistor    Vcc|---------------------------------| Vcc               |__________|
  typ 1-2k      CTS|                     .`          ------------      series resistor between header and chip UPDI pin on target PCB; I use 470 ohm resistors,
                Gnd|--------------------'                                so I can use a programmer that doesn't have a resistor built in.
--------------------

Awkward wiring to work with
--------------------                                   To Target device.
                DTR|               ,----------------------------------.   __________
    internal    Rx |--------------/                  | UPDI----\/\/\/--*-|UPDI      |
  Tx---/\/\/\---Tx |-------|<|---'          .--------| Gnd       4.7k    |          |
    resistor    Vcc|---------------------------------| Vcc               |__________|
  typ 1-2k      CTS|                     .'          ------------
                Gnd|--------------------'
--------------------





No internal resistor      diode method works if there's no resistor on the adapter (or if you jumpere over it) for resistors up to several k
--------------------                                   To Target device.
                DTR|                                  -----------         __________
 No resistor?   Rx |--------------,------------------| UPDI----\/\/\/----|UPDI      |
  Are you sure? Tx |----|<|------`          .--------| Gnd         \     |          |
 This is rare!  Vcc|---------------------------------| Vcc          \    |__________|
                CTS|                     .`           -----------    \Resistor of around a few hundred or more.
                Gnd|--------------------'
--------------------

         No resistor on target OR adapter - include a resistor in one of the three places shown below, whichever is easier to wire in.
         A few hundred ohms, I'd default 470. ideally not much more than that so you can use it with a target board that does have a resistor in series with UPDI

No internal resistor
--------------------                                   To Target device.
                DTR|                                  -----------         __________
 No resistor?   Rx |---------------------,--/\/\-----| UPDI--------------|UPDI      |
  Are you sure? Tx |--/\/\---|<|----\/\/'        .---| Gnd               |          |
 This is rare!  Vcc|---------------------------------| Vcc               |__________|
                CTS|                          .`      ------------
                Gnd|-------------------------'
--------------------


Resistor-based schemes - these have a narrower window of conditions under which they work reliably. They are not recommended unless there is something keeping you from using a diode cofiguration.


The PyUPDI classic:

                         4.7k resistor
No internal resistor
--------------------                                   To Target device.
                DTR|                                  -----------         __________
 No resistor?   Rx |--------------,------------------| UPDI--------------|UPDI      |
  Are you sure? Tx |--/\/\/\/\---`          .--------| Gnd               |          |
 This is rare!  Vcc|---------------------------------| Vcc               |__________|
                CTS|                     .`           -----------
                Gnd|--------------------'
--------------------

Very much like the classic, except for the possiility of a resistor on the target. Must be 470 or under on the target.

Resistance should sum to 4.7k
--------------------                                   To Target device.
                DTR|                                  -----------         __________
    internal    Rx |--------------,------------------| UPDI----\/\/\/----|UPDI      |
  Tx---/\/\/\---Tx |--/\/\/\/\---`          .--------| Gnd     =< 470    |          |
    resistor    Vcc|---------------------------------| Vcc          \    |__________|
  typ 1-2k      CTS|                     .`           -----------    \If resistor present, not more than 470 ohms.
                Gnd|--------------------'
--------------------

If the resistor on the target is much more than 470 ohms, you're gonna want to bypass it. Alternately, it may be easier to replace it with a 0 ohm resistor or bridge it with a piece of wire or even just a blob of solder, and do the classic pyupdi.


The resistor (if any) in serial adapter, and the one you add should total 4.7k.
--------------------                                   To Target device.
                DTR|                       ,--------------------------.   __________
    internal    Rx |--------------,-------'          | UPDI----\/\/\/- *-|UPDI      |
  Tx---/\/\/\---Tx |--/\/\/\/\---`          .--------| Gnd       >470    |          |
    resistor    Vcc|---------------------------------| Vcc           \   |__________|
  typ 1-2k      CTS|                     .`           -----------     \resistor of more than around  470 ohms - must be bypassed, replaced, or shorted.
or no resistor  Gnd|--------------------'
--------------------

If there's no resistor in the serial adapter, you can do it without any extrta componentys, thought you've got 4 wires involved instead of 3:

--------------------                                   To Target device.
                DTR|              .-----------------------------------.   __________
 No resistor?   Rx |-------------'      ,------------| UPDI----\/\/\/--*-|UPDI      |
  Are you sure?-Tx |-------------------'     .-------| Gnd       4.7k    |          |
 This is rare!  Vcc|---------------------------------| Vcc               |__________|
 OR resistor    CTS|                      .'          ------------    Excessively - but conveniently - sized resistor.  These were (incorrectly) popularized for the first few years of UPDI for ar
  bypassed      Gnd|---------------------'
--------------------



```

Why do we want a little bit of resistance (a few hundred) when doing the diode method? The classical response is "In case target and programmer are completely out of sync and try to drive the line in opposite direcctions" Note that on tinyAVR this isn't as much of a concern, as the output drivers on the UPDI/reset pin are so weak that it seems unlikely that they are capable of exceeding the maximum current per pin specification. Besides that, though, there's another reasonn: So that when (not if) you plug the connector in backward, but the target has external power too, there isn't a risk of the TX pin of adapter being damaged from trying to drive the positive supply rail low;

### Software

Choose "Serial Port and resistor or diode" from the Tools -> Programmer menu of a core which supports this upload method (megaTinyCore, DxCore, and soon, MegaCoreX), and select the Serial Port from the Tools -> Port menu.

Note that this does not give you serial monitor - you need to connect a serial adapter the normal way for that (I suggest using two, along with an external serial terminal application). This technique works with those $1 CH340 serial adapters from ebay, aliexpress, etc. Did you accidentally buy some that didn't have a DTR pin broken out, and so weren't very useful with the $2 Pro Minis you hoped to use them with? They're perfect for this. Although the CH340 parts have the lowest performance, the difference between parts is now quite small (this was not the case prior to the optimization leading up to the 2.3.2 release, before which the speeds ranged from 125 bytes per second to just over 1k.


### Upload and verify performance

  BAUD    |  FT232RL  kb/s   |   CP2102  kb/s   |   CH340  kb/s    |  HT42B534  kb/s   |
----------|------------------|------------------|------------------|-------------------|
115200    |   8.7 W /  8.8 R |   8.7 W /  8.8 R |  8.4 W /  8.5 R  |   9.0 W /  9.1 R  |
230400    |  16.6.W / 16.4 R |  16.3 W / 16.5 R | 14.6 W / 16.6 R  |  17.7 W / 17.9 R  |
345600*   |  24.3 W / 23.4 R |  23.2 W / 23.0 R | 22.5 W / 22.1 R  |    UNSUPPORTED    |
460800**  |       N/A        |        N/A       |       N/A        |   24.7W / 32.7 R  |
** HT42B534 was run using a 32-byte bloxk aiZe, as one will outrun the NVM controller writing at 460800 baud - I just had to see how it compared to the FT232RL. Both of them are running right up at the limit of the chip's ability to write data to the flash - and the FT232RL doesn't need any special measures taken and works with the tinyAVR parts too. On the other hand, the HT42B534 leads the pack at the (new as of 1.3.6) default of 230400 baud, and is dirt cheap (CH340-level prices).

For comparison, on the Dx-series parts (which are easier to use as test subjects since they have more flash, so uploads take longer and are easier to time. These numbers were taken using a 128k test image, which is an optimal situation.

Programmer      |  Read    | Write    | Notes                             |
----------------|----------|----------|-----------------------------------|
jtag2updi       | 6.6 kb/s | 5.9 kb/s | Running on 16 MHz Nano            |
Curiosity Nano  | 5.9 kb/s | 3.3 kb/s | Via avrdude - which is not ideal  |
Optiboot Dx     |10.6 kb/s | 6.9 kb/s | 115200 baud as supplied by DxCore |

Becausae of the smaller page sizes, ATtiny parts are slower to program, the two write numbers are for parts with 64 byte and 128 byte pages, respectively.

  BAUD    |  FT232RL  kb/s     |   CP2102* kb/s     |    CH340  kb/s      |  HT42B534  kb/s   |
----------|--------------------|--------------------|---------------------|-------------------|
115200    | 4.6,6.2 W /  8.8 R | 4.3,6.2 W /  8.8 R | 3.6,5.2 W /  8.5 R  | 3.6,7.2 W / 9.1 R |
230400    |7.5,10.0 W / 16.5 R | 7.1,9.6 W / 16.4 R | 4.9,7.8 W / 15.6 R  |  Errors out       |
345600*   |9.1,13.6 W / 23.4 R |8.5,13.2 W / 23.1 R | 5.6,9.5 W / 22.0 R  |  UNSUPPORTED      |
460800    |10.4,15.6 W/ 28.2 R |    Not tested      |6.0,10.4 w / 26.8 R  |  Errors out       |
* The CP2102 does not, by default, support any speeds between 256kbaud and 460800 baud - but a free configuration utility from Silicon Labs enables customization of the baud rates in each range of requested speeds (though unfortunately, you can't define those ranges). I reconfigured mine for 345600 baud for development of with the Dx-series parts, which don't work at 460800, and did not bother to set it back to factory settings just to fill in the table; I would expect to see approximately 10kb/s and 15kb/s write speeds and around 26kb/s read speed.


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
