# Trick for unbricking a classic AVR (inluding Uno/Nano or any chip released prior to ) which gets 0x000000 signature
## Cause
This only covers the situation when an attempt was made to "burn bootloader" and subsequently, AVRdude reports a signature mismatch. Enasble verbose output during upload, and you find that it is seeing a sigmature of 0x000000, and it fails immediately.

Depending on the hardware package and Arduino version it may have either done part of the bootloading process and then failed, or have appeared to succeed... Except now you can't do any more programming of any sort!

You will also have this problem if you take a chip that had been used in, for example. an Uno, and then want to use it with

If the failure occurred during a bootloading attempt *AND THE FIRST ATTEMPT GAVE DIFFERENT OUTPUT*, 11 times of of 10, this indicates that you've just set it to use an external clock, or a crystal, but either your external clock is actually just a crystal good crystals come in the same 5070, 5032, or 3225 packages as oscillators, with the same shiny metal cover, and (so they claim) a part number on the top. Sometimes the part number is readable, but often it's not. (Solder readily wets the top of the case too, so sloppy hand soldering can irrevokablly obscure any markings on it (as can intentional desoldering method of resting wide iron tip on top of case to melt the otherwise inaccessible pads while using an iron. I don't recommend that method, but I've done it. Don't bother trying to remove the solder with desoldering braid to read the top, you'll never be successful).

Regardless of whether you set it to crystal or extermnal clock, it ain't getting a clock signal from it. It's system clock is hence ZERO. Which makes the requirement that SCK for programming be lessthan 1/6th of the system clock problematic.

## Solution
There are two approaches to rectify the problem, largely depending on what your goal of that part is

### You want to use it with the clock source you set it for
In this case, the problem is either a defective or failed crystal (it happens - it's not common, but it's far from unheardof), a bad connection, or a failure to observe basic design guidelines for high frequency signals.

#### Connections
Check for shorts between crystal pins and anything nearby; small SMD caps used as the loading caps can sometimes have a solder bridge UNDER the capacitor - it's easy to find which side of the crystal is shorted, determine visually that there's no bridge that you can see - then you go for the cap. Check that both sides of the crystal have continuity to the pin on the chip. Double check that they're the right pins. Examine the solder joint carefully - you sure it's good? I've seen solder joints that are not making contact... until you press down on the pin with the multimeter probe, pushing it into contact so that it looks okay. When baffled, I have sometimes tried reflowing solder joints that could explain the problem "defensively".

#### Capacitance and layout
Are your load capacitors correct? They should be similar to the capacitor's load capacitance specification (there is a formula forthis, and it's more complicated than that). That's pF - probably the omnly place you will use capacitors with values expressed in units of pF.

Is the crystal on the same side of the PCB, with nice short traces between the microcontroller, crystal amd loading caps? If not, that can prevent it from oscillating.

Are you using solderless breadboard? Solderless breadboard has an incredibly high stray capacitance. People often report having to use much smaller loading caps - or even none at all - in order to make their crystals work on breadboard.


Active oscillators (external clocks) have the same problem with length of traces. You also need to supply them with power. Did you connect it the right way around? Unlike crystals, they are polariozed, and connecting them backwards will destroy them. They also need their own decoupling capacitor, generally spec'ed at 0.01uF (much lower than a microcontroller's decoupling cap wants to belocated as close as possible to the oscillator), which should be located as close to the oscillator's power pin as possible.

### If you didn't want to use a crystal or external clock at all
In this case,where you unintintionally softbricked it, you simply need to provide a square wave with approximately 50% duty cycle, somrething like 2 MHz would be ideal. to the XTAL1/CLKIN pin (this is generally the same pin) while you reprogram it.


## Unbricking
Disconnect all other devices form the XTAL1 opr CLKIN pin (likely the defective crystal, or dead oscillator or wwhateverelse 0 but couldbe anything if didn't meant to use that opin  forthispurpose.

You can do this with the [ArduinoAsISP_CLOCK sketch](ArduinoISP_CLLOCK?ArduinoISP_CLLOCK.ino) in place of Arduinio as ISP. It outputs an 2 MHz square wave on pin 8. Connect the ISP header, prepare to boootload, connect the clocik line to CLKIN/XTAL1, buurn bootloader with correct options, and you're done, the extermnal clock can and should be disconnected. It is common to use imporovised and tempeorarty connections to the CLKIN/XTAL1 pin - a pogopinm weith a wire soldered to either it or a matching receptacle is probably what I would do. You only needto make contact for a few seconds/
