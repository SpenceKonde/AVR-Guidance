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

When making a 3-pin cable, we recommend this pinout:

UPDI - GND - VCC    (or equivalently, VCC - GND - UPDI).

For the simple reason that it minimizes the chance of damage if it is plugged in backwards. 


# Serial-UPDI: UPDI programmer from serial adapter (recommended)
As of DxCore 1.3.0 and megaTinyCore 2.2.6, it is now possible to use a serial adapter and a schottky diode! (or a 4.7k resistor - but the diode works better)
As of megaTinyCore 2.3.2 and DxCore 1.3.6, these are much faster than jtag2updi, and are the recommended method of programming.

### Required components
1. A USB serial adapter These can be had for as low as $1 on ebay and aliexpress based on the CH340G, slightly more for CP210. Ideally, you want to dedicate a serial adapter to this purpose for ease of use, rather than havign to connect and disconnect things every rtime you want to use it. 
1. a fast signal schottky diode such as a 1N4148 or any of many others (recommended) or a resistor (see below to figure out value)).
1. 1 resistor, a few hundred ohms - 220 to 1k, even 2k is fine (may not be needed). 
2. A few jumper wires.

### Serial adapter requirements
Almost any serial adapter can be used for pyupdi style programmer, as long as you take care to avoid these pitfalls:
1. The FTDI FT232, (both the genuine ones, and the fakes) are by default configured to use less CPU time by polling the adapter less often. For UPDI programming, the performance implications are severe. You can easily fix this (at least on windows - I'm not sure if the problem even happens on Linux): Open device manager, under Ports (COM and LPT), locate the FTDI adapter. Right click -> properties. Click the Port Settings tab, and then the Advanced button. In the middle of left hand side, there is an option called "Latency timer", likely set to 16ms. Set it to 1ms). Click OK enough times to leave the dialog. you'll see the adapter disappear and reappear as the change is applied. Configured properly, the FT232RL has spectacular performance. Configured improperly, the performance is downright abysmal.
2. MOST SERIAL ADAPTERS ALREADY HAVE THEIR OWN RESISTOR in series with Tx! Typically between 1k and 2.2k; If using the recommended diode method, you need only add the diode, not the second resistor. If you aren't going with the diode method for some reason, use a resistor such that the total is 4.7k (you'll need to measure it with multimeter unless you can follow traces to the resistor and read out the code (the three number codees - first two numbers are the most significant digits, and the third is the number of zeros they are followed by - 222 is 2200 - 2.2k, 471 is 470, (which of course means that 101 means a 100 ohm resistor, and 100 means a 10 ohm one!)).  There are three approaches to measuring all wuith one end of multimeter clipped to Tx. Either look up the pinout of the serial adapter chip online and just measure resistance to that pin, check all the pins on the chip. Many may have very high resistance, but only one will have a value of a few k or less, just go for the ends of the resistors, expecting one to have a 1-2.2k to Tx on one side, and 0 on the other/
(if you actually encounter one without a resistor between Tx and the chip, let me know! I only know of one design circulating in the wild: it's the CH340G on a green PCB with a 3.3v/5v switch and microUSB port; that board appears to be someone's first design - which they have nonetheless been producing for years without revision. The design has 2 very desirable features - voltage switch, and micro USB - and at least 4 severe design flaws.)
3. Some serial adapters have a dedicated LED to indicate Rx. While some fancy chips have an I/O pin that drives the RX led (the FT232, for example), a cheap adapter with an RX LED may have just put an LED and resistor on the RX line (in fact that's what that above mentioned green boards with switch and microusb port did).. The load from an LED on the UPDI line will overwhelm any signal from the target and prevent communication  (a LED on TX is fine - the adapter has plenty of drive strength.)

I recommend the bog standard CH340G as the go-to serial adapter for several reasons:
* They are dirt cheap and readily available on ebay/aliexpress/amazon. 7/$5 shipped on aliexpress for the most basic design (the black kind with the stupiud voltage jumper on the end. The wider ones with rounded corners and ENIG surface treatment are no better electrically, though the build quality has to be better (the first kind you should plan to solder the USB connector onto more solidly), and are bulkier. The better kind are what you find on ebay when you search for "CH340 6pin" - you want the black ones with the voltage switch. Until, that is, I bring my own design into production - it'll have voltage switching done *right* and a micro USB connector instead of a stuipid full sized USB-A, full-sized 1117 3.3v regulator, and a color-coded power light to indicate the current voltage setting, and a general absence of gross design flaws.
* The bare chips are dirt cheap and readily available - and also dead simple to design with. Anyone, even the clown who designed those green boards I've been badmouthing, can do it (it looked like someone's first board, frankly). Heck, a working CH340G serial adapter was the first board I designed, too (at least I didn't try to sell my first version!).
* They are - at most - a half second slower to write a full flash image with. For the price of one real FT232RL adapter, you could have half a dozen CH340's.

And because the alternatives all have problems:
* Most of the FT232RL boards for sale are fake. I have several on order to compare with my real one. If they turn out to perform comparably, I will update this (though even the fakes are more than twice the price of a CH340)
* The CP2102 doesn't support the top speed for Dx-series (345600 baud) - or, if you configure it to support that with the utility from Silicon Labs, you lose support for 460800 for tinyAVR.

* The HT42B534 has a variety of unusual attributes, some good, some bad.
  * It doesn't support arbitrary baud rates - no 345600 for Dx-series, or anything between 256 and 460.
  * On the one hand, it uses standard windows CDC drivers; that's not a bad thing - not that I've ever found keeping serial drivers installed for other adapters to be a terrible burden. Unfortunately, they don't work very well. Setting it to a speed it doesn't like not only doesn't have the desired effect, but the attempt blocks rather than terminating gracefully. serialupdi has a fairly short timeout builtin and errors; many terminal programs will just hang until you restart them or unplug the serial adapter and plug it back in. Sometimes it stays behind in device manager after unplugging it, particularly after it hangs.
  * Modem control inputs are wrong: All show off until any one changes state, then all show opposite of actual value. Inverted behavior continues until next reset.
  * The chip (though not the available adapters) have a VDDIO pin to set output voltages, ie, a built-in level shifter. CH340 can only do 3.3V-5V, HT42 can do 1.8-5v. It would be a very nice feature if it was exposed in breakout boards;
  * They almost CH340 cheap.
  * The latency is significantly lower than other serial adapters. Earlier during the spring 2021 optimization effort, after optimization for the "simpler case" of writing to a Dx-series (it is simpler in that there are larger pages, hence larger blocks of time spent , the speed of uploads was determined in large part by latency, rather than raw speed. This version was

Before the optimizations to serial-updi, it absolutely crushed all of the competition. But as I improved performance of the other parts, they could catch up, while the Holtek chip had little room between it's head and the physics-imposed performance ceiling. I am very surprised that the write performance on tinyAVR wasn't better. At some point I will retest - but this all isn't why I don't recommend these parts. There are two issues:
1. The modem control inputs are backwards! At power on, they all show as off. Then, the moment one of them changes state the first time, they all show inverted logic - when an external device asserts (grounds) one, it shows as off, otherwise it shows as on. Opposite of how every other manufacturers' adapter behaves.
2. They have far and away the highest incidence of driver hangs that required the board to be replugged. If you try too high of a baud rate. If you try too low of a baud rate. It you look at them crosseyed. If you look at them without crossing your eyes. If they were actually still faster, I could overlook this (I will overlook a lot to get faster uploads)... but on the Dx-series, they don't support 345600 baud, so the best they can do is 460800, but with --blocksize 32 to slow it down so it doesn't out-write the NVM controller. Whereas on the tinyAVR parts, above 115200 baud, they error out (my theory is that they try to send the next command while the chip is still writing the flash page - on the Dx-series, it programs a word at a time (so it is sustained write speed that breaks - bit s get dropped when the controller reaches 24 bit-periods (1 data word) ahead of the target's NVMCTRL), whereas on the tinyAVR, it is the latency betweeen sending the write page command, and when it starts trying to write again.


### A note on breakout boards
Some tinyAVR and other UPDI-based part breakout boards have an on-board resistor. Sometimes this is a 4.7k one. That is NOT appropriate. I was part of the problem for a while. I think the original mistake came from people conflating the pyupdi resistor with a generally appropriate one. When I started megaTinyCore, my early collaborator was making hardware with a 4.7k resistor; I assumed he was doing it right. While this does work with dedicated programmers, including jtag2updi, it doesn't work with serial UPDIO. It will work with dedicated programmers like jtag2updi, as long as they don't have their own resistor. Suffice to say, for a time it was a very common belief. I use 470 ohms now, but I can't find fault with a design over it not having one at all.

### Connections:
* Vcc, Gnd of serial adapter to Vcc, Gnd of target
* 4.7k resistor between Tx and Rx of adapter (many adapters have built-in 1k, 1.5k, or 2.2k resistor in series with Tx; these should use a proportionally smaller resistor)
  * For better results, a smaller resistor (that built-in one on most adapters, mentioned above, will do perfectly here) and a small schottky diode (band towards Tx, other end connected to Rx) can be used (use a "small signal diode" - larger general purpose diodes may have properties that make them less suitable for this) The diode substantially widens the tolerances of this programming method, and significantly improves reliability.
   * My top pick here is the BAT54C,235; it's in a tiny SOT-23 package (it's 2 diodes both weith the "band" towards the pin thats' alone on one side) Why? Because, assuming your serial adapter has the pins on 0.1" header, and TX and RX are next to eachother (both extremely common) the the diode fits right in beteeen them. and with no lead that could later fatigue and break the result is less likely to be damaged by rough handling. Then if I want to more ovbviously mark it as a UPDI programmer, I might cut off the Tx, DTR and CTS pin; always remember that you can get serial adapters for a buck a piece on ebay.
* Rx of adapter to UPDI pin of target. A small resistor (under 1k - like the 470 ohm one we generally recommend) in series with this is fine.


```
USB Serial Adapter
With internal 1-2k resistor on TX
This is the case in 90% of USB serial adapters.


Ideal:
internal resistor in adapter: 2.2k >= Ra
resistor on target:   2k >= Rt >= 100 (100-470 rec.)

--------------------                                 To Target device.
                DTR|                                  __________________
    internal    Rx |--------------,------------------| UPDI---\/\/---->
  Tx---/\/\/\---Tx |-------|<|---'          .--------| Gnd    470 ohm (100 ~ 1k)
    resistor    Vcc|---------------------------------| Vcc
  typ 1-2k      CTS|                     .`          |__________________
                Gnd|--------------------'             If you make a 3-pin connector, use this pinout
--------------------





Near Ideal:
Yes internal resistor in adapter: 2.2k >= Ra
No resistor on target

--------------------                                 To Target device.
                DTR|                                  __________________
    internal    Rx |--------------,------------------| UPDI----------->
  Tx---/\/\/\---Tx |-------|<|---'          .--------| Gnd
    resistor    Vcc|---------------------------------| Vcc
  typ 1-2k      CTS|                     .`          |__________________
                Gnd|--------------------'
--------------------



Yes internal resistor on adapter
Yes resistor on target: small, under 2k

Also works great, convenient if still using jtag2updi without resistor built into it. Resistorss should sum to less than 4.7k, preferalby much less

--------------------                                   To Target device.
                DTR|                                  __________________
    internal    Rx |--------------,------------------| UPDI----\/\/\/---->
  Tx---/\/\/\---Tx |-------|<|---'          .--------| Gnd     < 2.2k
    resistor    Vcc|---------------------------------| Vcc
  typ 1-2k      CTS|                     .`          |__________________
                Gnd|--------------------'           Series resistor between header and chip UPDI pin on target PCB
--------------------                                I use 470 ohm resistors.

Yes internal resistor on adapter
Yes resistor on target: several k or more:
This will often by 4.7k: it must be bypassed, replaced with a smaller one or shorted out.


--------------------                                   To Target device.
                DTR|               ,----------------------------------.
    internal    Rx |--------------/                  | UPDI----\/\/\/--*-
  Tx---/\/\/\---Tx |-------|<|---'          .--------| Gnd       4.7k
    resistor    Vcc|---------------------------------| Vcc
  typ 1-2k      CTS|                     .'          |__________________
                Gnd|--------------------'
--------------------



No internal resistor on adapter.
Yes resistor on target >= 100 ohms and not more than a few k.

--------------------                                   To Target device.
                DTR|                                  __________________
 No resistor?   Rx |--------------,------------------| UPDI----\/\/\/------>
  Are you sure? Tx |----|<|------`          .--------| Gnd     > 100
 This is rare!  Vcc|---------------------------------| Vcc     < 2.2k
                CTS|                     .`          |__________________    Resistor of around a few hundred to a few k
                Gnd|--------------------'
--------------------


No resistor on target OR adapter:

Include A resistor in ONE of the three places shown below, whichever is easier to wire in,
the first and second positions, with nothing between Rx and target UPDI, are slightly preferable

Value of resistor should be a few hundred ohms, I'd default 470.

No internal resistor
--------------------                                   To Target device.
                DTR|                                  __________________
 No resistor?   Rx |---------------------,--/\/\-----| UPDI---------------->
  Are you sure? Tx |--/\/\---|<|----\/\/'        .---| Gnd
 This is rare!  Vcc|---------------------------------| Vcc
                CTS|                          .`     |__________________
                Gnd|-------------------------'
--------------------


Resistor-based schemes - these have a narrower window of parameters under which they work reliably.
If you look at the UPDI line on a 'scope while it is malfunctioning, you will see that sometimes
the voltage is not going all the way down to ground when one side tries to assert it.

They are not recommended unless there is something keeping you from using a diode cofiguration.


The PyUPDI classic:

                         4.7k resistor
No internal resistor
--------------------                                   To Target device.
                DTR|                                  __________________
 No resistor?   Rx |--------------,------------------| UPDI---------------->
  Are you sure? Tx |--/\/\/\/\---`          .--------| Gnd
 This is rare!  Vcc|---------------------------------| Vcc
                CTS|                     .`          |__________________
                Gnd|--------------------'
--------------------

Very much like the classic, except for the possiility of a resistor on the target. Must be 470 or under on the target.

Resistance should sum to 4.7k
--------------------                                   To Target device.
                DTR|                                  __________________
    internal    Rx |--------------,------------------| UPDI----\/\/\/------>
  Tx---/\/\/\---Tx |--/\/\/\/\---`          .--------| Gnd     =< 470
    resistor    Vcc|---------------------------------| Vcc         \If resistor present, not more than 470 ohms.
  typ 1-2k      CTS|                     .`          |__________________
                Gnd|--------------------'
--------------------


If the resistor on the target is much more than 470 ohms, you're going to want to bypass it. Alternately,
it may be easier to replace it with a 0 ohm resistor or bridge it with a piece of wire or even
just a blob of solder, and do the classic pyupdi. Note that using a diode will work with resistances oin the target
that are too much for it to work using a resistor.


The resistor (if any) in serial adapter, and the one you add should total 4.7k.
--------------------                                   To Target device.
                DTR|                       ,--------------------------.
    internal    Rx |--------------,-------'          | UPDI----\/\/\/- *--->
  Tx---/\/\/\---Tx |--/\/\/\/\---`          .--------| Gnd       >470
    resistor    Vcc|---------------------------------| Vcc
  typ 1-2k      CTS|                     .`          |__________________     \resistor of more than around  470 ohms - must be bypassed, replaced, or shorted.
or no resistor  Gnd|--------------------'
--------------------

If there's no resistor in the serial adapter and the target happens to have a 4.7k resistor, you can do it without
any extrta components, though you've got 4 wires involved instead of 3:

--------------------                                   To Target device.
                DTR|              .-----------------------------------.
 No resistor?   Rx |-------------'      ,------------| UPDI----\/\/\/--*---->
  Are you sure?-Tx |-------------------'     .-------| Gnd       4.7k
 This is rare!  Vcc|---------------------------------| Vcc
 OR resistor    CTS|                      .'         |__________________
  bypassed      Gnd|---------------------'       Excessively - but conveniently - sized resistor.
--------------------                        These were (incorrectly) popularized for the first few years of UPDI



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
** HT42B534 was run using a 32-byte bloxk size, running with finite block size resulted in successful transfers for other parts, though the threshold block size varied - but a massive decrease in overall speed, similar to 115200 baud., as one will outrun the NVM controller writing at 460800 baud - I just had to see how it compared to the FT232RL. Both of them are running right up at the limit of the chip's ability to write data to the flash - and the FT232RL doesn't need any special measures taken and works with the tinyAVR parts too. On the other hand, the HT42B534 leads the pack at the (new as of 1.3.6) default of 230400 baud, and is dirt cheap (CH340-level prices).

For comparison, on the Dx-series parts (which are easier to use as test subjects since they have more flash, so uploads take longer and are easier to time. These numbers were taken using a 128k test image, which is an optimal situation.

Programmer      |  Read    | Write    | Notes                                    |
----------------|----------|----------|------------------------------------------|
jtag2updi       | 6.6 kb/s | 5.9 kb/s | Running on 16 MHz Nano                   |
Curiosity Nano  | 5.9 kb/s | 3.3 kb/s | Via avrdude - which is likely not ideal  |
Optiboot Dx     |10.6 kb/s | 6.9 kb/s | 115200 baud as supplied by DxCore        |

Because of the smaller page sizes, ATtiny parts are slower to program; the smaller the pages, the slower the data rate - but the time for programming the entire flash is still less for smaller parts because there is less data to write. The two write numbers are for parts with 64 byte and 128 byte pages, respectively.

  BAUD    |   FT232RL  kb/s     |   CP2102* kb/s     |   - CH340  kb/s      |  HT42B534  kb/s   |
----------|---------------------|--------------------|----------------------|-------------------|
115200    | 4.6, 6.2 W /  8.8 R | 4.3,6.2 W /  8.8 R | 3.6, 5.2 W /  8.5 R  | 3.6,7.2 W / 9.1 R |
230400    | 7.5,10.0 W / 16.5 R | 7.1,9.6 W / 16.4 R | 4.9, 7.8 W / 15.6 R  |  Errors out       |
345600*   | 9.1,13.6 W / 23.4 R |8.5,13.2 W / 23.1 R | 5.6, 9.5 W / 22.0 R  |  UNSUPPORTED      |
460800    |10.4,15.6 W / 28.2 R |    Not tested      | 6.0,10.4 w / 26.8 R  |  Errors out       |
* The CP2102 does not, by default, support any speeds between 256kbaud and 460800 baud - but a free configuration utility from Silicon Labs enables customization of the baud rates in each range of requested speeds (though unfortunately, you can't define those ranges). I reconfigured mine for 345600 baud for development of with the Dx-series parts, which don't work at 460800, and did not bother to set it back to factory settings just to fill in the table; I would expect to see approximately 10kb/s and 15kb/s write speeds and around 26kb/s read speed.

#### Observations on Speed
The following is based on the numbers above, test data not included, as well as examination of oscilloscope trace. Note that the 460800 baud data for the Dx-series is not directly comparable as it uses blocksize to artificially slow the writes to prevent overrunning the buffer which the chip empties at a constant rate.

Approximately, write time can be modeled as:
T = ceil(size / pagesize) * C + size / (baud in bits/sec / 12 bits/byte)
where C in turn can be modeled as:
C = (c * n) + d/(baud / 12)

12 bits to 1 byte because the UPDI protocol uses 1 parity bit and 2 stop bits.

The same relationship holds - with a different constant - for read and write with the size of blocks read substituting for page size.

`d` is the overhead in bytes involved in setting up each page; it depends on the series of the part in question.

`c` is a charachteristic time depending on the serial adapter - the USB Latency. This is not actually a single consistent, constant value; potential values of it appear to be quantized and certain usb latency pauses within the upload process are of different durations.

`n` in a the number of separate USB transactions required to set up each page write, a function of the procedure used by the programming tool. Most of the optimization involved reducing this number; it is 2 for reads and Dx-series writes. For tinyAVR and megaAVR it is 4 when writing.

Monitoring the transactions on a scope, one can pauses with small exchanges p
(small overhead)->(latency) repeated n times per page, followed by a write lasting a period of time proportional to (pagesize + d)

Within the region of interest, baud is large relative to d, so C is nearly constant

At low baud rates, `T` tends towards a inverse relationship with baud.
At high baud rates, `T` tends toward a function of size, NVM version, and serial adapter only. Overcoming this limit requires reducing the number of USB transactions per page written, or block read.

T<sub>read</sub> is the same on all parts; an unimplemeneted change could push it a bit lower by bringing N all the way down to 1 from 2. The practical impact, however, is very small; after the end of the perfomance enhancement push, T<sub>read</sub>, reading in 512b blocks is dominated by the second term.

For the Dx-series parts as shown in the above table, with the full suite of optimization, T<sub>read</sub> = T<sub>write</sub> (the same change is likely possible here; the improvement is maybe 10%)

For tinyAVR, T<sub>read</sub> is the same as Dx, but T<sub>write</sub> is far worse - not only is pagesize smaller but `n` 4 instead of 2 so `C` is twice as high. At 460800 baud, the second term is about a third the magnitude of the first for write (and the tinies will accept writes at 691.2 kbaud). The important thing to keep in mind is that these chips still program in seconds - and the page write is being committed during the time that one of these pauses is ongoing.

# jtag2updi - UPDI programmer from a Nano or Pro Mini
A completely different approach is to use an Arduino sketch is available to turn ATmega328(p)-based Arduino’s, like the Arduino UNO and Nano, into a UPDI programmer (it does not work on boards based on other parts, like the 32u4 (Micro/Leo) or any non-AVR board). The following steps show how to make one of these programmers. While this was previously our recommended programmer, SerialUPDI is now more reliable, is cheaper to build, and performs better than jtag2updi, even on the tinyAVR parts whose small page sizes should give the advantage to jtag2updi. A major limitation of jtag2updi is that it is limited to approximately half of the used baud rate, since it must retransmit to the target through a software serial port.... and that baud rate in turn is limited to 115200 baud, 8 bit

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
