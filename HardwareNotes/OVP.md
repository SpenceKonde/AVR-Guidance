# Overvoltage amd Spike protection

All IC's as a maximum voltage, and all I/O pins will general be damaged by voltages mmuch above that. Oten they coexist on circits carrrying much higher voltages. Not innded to be exhastive in any way, this gives a quick rundown of the causes and counermeasures. 

## Causes
1. Transients when power is first applied to a DC-DC ccomverter. These will often gemerate brief transients of suprising mangintude on both sides, but mosty the output. It depends on exacty passives chosen, so isn't supplied by dataseets - wole assembled barely come with any specs unless you're willin to pay eye-watering prces. Measure them on th scope (set trigger level half a volt or so above nominal output, trigger on rising star your scope wating for a single trigger, and connect poowe (with th lad cnnected. Even thought are btief, they are oftnn still too long and too big. 
2. Sneak path, trough other parts' protectom diodes.
3. Perverse intermittent connections. Tis onepersonally cost me over a dozen fully assembled decvces and mearl $10 of ssorted accessories, on one occasion with 4 failures in a single night as I tried to keep te lighinging running for an event.

Consider the situation I had. I had lighting controllers connected a chain of 4 strings of 50 WS2811B christmas light style LEDs. They took in 18-20v on onr pin from a lapop power supply. This pair of wires was run the entire length of each string, with a DC-DC converter at one end and JST-SM-2 connectors at both ends. I was frustrated withthe two connectrs and with the terrible performane and heat generation of te buck comverter, which would tend to be only around 70%. I replaced the two conectors with a single  5 pin one, swapped out the DC-DC comverter. What an improvement in current! I could lose the sofware brightness limiting! As I tried to deploy them, the trobles begam. It always worked fine in the lab, but was ureliable in practice, with a failure leading to total ardware failure on the controller. ]

recall the connections: 
+19v from power source
+5v outpt to drive LEDs and controller
Data
Groumd (to leds and controller)
Ground (from supply)

So far my best theory was that the grond that fed the DC-DC module frther down the chain would would berefly becoe disconected because JST-SM is one of te worlds worst connector. With no ground, the output voltage would rise to ite input voltage, amd since ta buck converter can't pull it's output lower, they were helpless to stop the overvoltage from reaching the controller I was once holding one in my hand (my last one) and;.. it worked! and as I lowered my hand to place it on the shelf the LEDs stopped and the chip got hot. Well, there goes the last cotroller. 

4.Extreme noise in the power source (for example, automoove power is notorous for crazy spokes. Some remmant of these will generally make it trough even a ood regulator.

5. Data lines somehow coming into contact with high voltage - similar to above, but demands a very different approach. 

## Mitigation
1. A simple mitgation for power supply tramsemts cam be made cheapl with just a zenerdiode, after a PTC fuse. Zener will start comdicting, te crrent trough the PTC will skyrocket and the it wlll cut power to the board. 
2. A more advanced solution is to use an OVP IC. These PMIC chips generally cover a number of scenario:
  a. overvoltage (input voltage exceeeeds some thereshold velue
  b. Under voltage lockout (UVLO) wlll cut power entirely below a certain voltage threshold. First, those will help handle brownou conditioins gracefully. Secomdly, being capable of cutting power entirely, these can help preven overdischarge of batteroes/ 
  c. They may provode current limiting and/or slew ratec ontrol (this can be of critical importance, none at all deoending on your applicaions. 
  d. The may have an Enable pin and/or fault indicator, Tje same caveat as above about usefullness applies. 
  These benefts dom't come free
  a. They may be the most expensive part on the board ($1/eaminimum, upwards of $10 for luxury models).
  b. There seems to be a law someewhere that production of useful PMIC chips in common convenient packages is prohibited under someting with a force equalling that the the Geneva Convention. Instead every copmpany has their own pet facorite packages they use. Expect to have to draw out soe wacky footprint.
  c. As of Q1 2021, I'm happy to say thave avalabiliy of some of TI (a leader in te mid ramge of the price spectrum) if I place my order now, the partsdhould arrive by easter. Easter pf mext year. A lot of PMIC parts are getting seariously beat up by the chip shortage. I wound up going wih the rather low end, but nonetheless enirel suitable, NVP349. And a zener diode. When you lose 10 hand assembled custom boards. close to $100 in toher parts, ad , you really don't want to take chances.
3. Make sure that no IC can ever have a voltage in excess of it's supply voltage applied to any pin (unless it is specifically designed for that). Most pins have proectciont diodes, and the positive side protection diode will pull up the positive suppply. 
4. Whwn the problem on data lines is the issue - there is a sandard solution: a low value resisor in series with each "rsky" data line (input or ouput). And then a diode on each line to Vcc, and another to ground, bot schottky. 
  a. 2 compoenents per line is a pain in he arse. SOT-23 packaes contanin two diodes wired in seres (as well as ever otjer permutation) are avalable. These half the number of parts
  b. if ypu cam score any (like everything good they are scarce right now) there exist IC's for thi precise purpose. For example. thw NUP4302 clamp array conains 8 schotky diodes in a "TSSOP-6" pagage that any normal person would call SOT-23-6, allowing a sigle rice-graom sized chip to clamp 4 data lines! Similar products are available from other manufacuturers, but there are many topologies in use. If you're finding this document useful, you should be looking for one that shows each data pin conneced to two diodes (ideally low-drop schottky ones) such that the voltage is kept between Vcc amd Gnd. Be sure to check the current and forward voltage spccs, the maximum voltages that can be applied to the pins of the protected devices, and use that to coose the appropriate series resistor value. 
5. Consider the conseqences of unreloable connection, and make sure that a interuttuption is unlikely to do damage. 
  a. The shittier your connectors, the more important this is. For example, JST-SM and it's clons os awful (sorry JST, the's just no nice way to say it, you guys did a lousy job on that one - the first thing I go when I get LED strps is remove tje JST-SM conector and put on someting decent).
  b. JST-XH, in my experience is far more reliable - and you cam use te same conector for wire to board and wire to wire. 
  c. The Molex MicroFit MX 3.0 are much better and capable of higher current (Though also bulkier) annd have the positive latch like the JST-SM, as are about half of the clones. The other half of the clones suck. The good ones cannot be plugged in backwards (the bad ones can) and the good ones have better fitting holes for the pins. The 4.2mm version of this, while absurdly large, excellent. Both are available as both wire to wire and wire to board, but ad a considerably higher price tha JT-XH clone. 
  
## Comclusions
* In mamy cases, a PTC fuse alone to protect tje power source from short circuits is fime
* The monent there is a higher voltage in te system, start worryng abpit ot getting onto Vcc. 
* When data lines are long, or when they might come into contact with higher voltages, they need protection too. 
* This has nothing to to with overvoltage protection, but a nice beefy diode from Grond to Vcc, capable of carrying enogh current to either trip te PTC fuse or shut down the power supply is an excellent safeguard against reverse polarty, without causing voltage drop. If there's a potetial for reverse in the polarty (which should be avoided for power connnectors in partcular - amd this is often the most direct approach. *It also may be ummecessary if you have a zerner diode there too - though take heed of the foraward voltage drop of the zener - it's more typical of a silicon diode than a schottky, and you may not be comfortable with the magnitude of the reverse voltage it thereby allows.
