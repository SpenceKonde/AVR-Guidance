# DC-DC converter modules
A DC-DC converter converts DC power at one voltage to DC power at a different voltage. While by this definition, you could say any voltage regulator was a DC-DC converter, this document will be limited to so-called "switching" DC-DC converters. These use an inductor, a diode, and a MOSFET, plus input and output capacitors. The diode and the MOSFET act as switches, and there are several paths the current can take, some of which lead through the inductor. A current in an inductor strengthens a magnetic field (which takes energy) in one phase of the cycle, and the magnetic field sustains the current (releasing the energy) in the other phase. Depending on the topology, you get either a buck converter or a boost converter. A buck converter can only produce a lower voltage than the input, while a boost converter can only produce a higher one. Respecting conservation of energy, the input current will differ such that the power (I (amps) * V (volts)) = P (watts)) of the input is equal to the power of the output plus a factor to account for the efficiency of the converter.

## Quick review
### There are tons of resources explaining this
There are lovely animated diagrams showing how these work in countless places across the internet. The purpose of this work is not to reproduce those resources; The wikipedia article is a good description of the fundamentals.

### Buck Converters
These are cheaper, and more common. It's easier to turn a high voltage into a lower one than the other way around. In a buck converter, the diode is connected between ground and the inductor's input side. When "on", the MOSFET lets current flow from the input supply, through the inductor, and then to the load, (with capacitors on the input and output of course, for stability). Since an inductor will oppose a change in the current (either storing the energy in it's magnetic field, or releasing it accordingly), this charges up the inductor. When the switch turns "off", to maintain the current through the inductor, current flows from ground through the inductor, releasing that energy, and the output voltage is determined by the dutycycle (and inefficiencies in the system). The input current is hence lower on average, because part of the time the current is coming from ground instead.

### Boost Converters
These connect the inductor between the input and ground when turned "on" in order to charge the inductor's field. When it is turned "off" the current then flows from the positive supply through the inductor through the diode and then into the load (and output capacitor) - notice that where before the inductor had to "pull" current out of ground, here the inductor has the input supply on it's side. Input current flows continually, but only into the load when the switch is off. The input current is hence higher on average - since part of the time it's being used just to charge up the inductor.

### Buck-Boost converters
As you might expect, these allow you to either increase or decrease the voltage - unfortunately, doing that takes two inductors and two MOSFETs. That both makes it more expensive, and potentially reduces efficiency. These are essentially a boost and buck converter back to back.

### Switched Capacitor converters
A completely different strategy: You can imagine charging a capacitor up, then flipping it around and putting it in series with the power supply to double the voltage, or charging two capacitors in series, and then reconnecting them in parallel to halve the voltage. Similar manipulation can also generate negative voltages. They are often called "charge pumps" These are much simpler conceptually - and generally cheaper to build for small voltage changes - but they are far less efficient, and in and of themselves, only achieve integer multiplication or division of voltages, minus any inefficiencies. With ideal components (zero resistance inductors, charging capacitors with no ESR or ESL, diodes with zero forward voltage drop, no leakage anywhere, and so on), a buck or boost converter approaches perfect efficiency. A charge pump can only do that for certain ratios of input and output voltage. A very simple charge pump can be made with anything that generates PWM, an output capacitor, a "flying" capacitor, and a pair of diodes - you can do this with a microcontroller pin (put a resistor on it to limit the current and prevent damage to the pin). The flying capacitor is placed between the PWM source, and a diode to the positive side of the output capacitor, with the other diode between the positive input supply and the other side of the flying capacitor. The voltage on the output capacitor will be significantly higher - up to the supply voltage plus the peak-to-peak voltage of the PWM, minus the drop of the two diodes.
Neglecting the drop from the diodes, while the pin is low, the capacitor charges up to Vcc. Then the pin jumps up to Vcc - the other side of the capacitor has to follow, since the voltage across the capacitor resists change. Charging capacitors from other capacitors is easy to imagine - but it's also not terribly efficient. Specifically, while it's reasonably effective at producing the expected voltages under no load, the efficiency plummets as the load current increases. Accordingly, "cap switching" or "charge pump" converters are only used when only very low current is required. Some versions of capacitor-switched converters are used to generate exceedingly high voltages - but only at vanishingly small current and miserable efficiency.

An example of a more mundane application - which was ubiquitous in early flash memory devices, which required a +12v rail to power the "erase" cycle, but otherwise operated at lower voltages. Chips like the MAX662A, a regulated 5v-to-12v switched capacitor converter were used until technology had advanced such that the charge pump could be integrated with the other components. That chip is still in production today, though volumes have declined and some second sources have exited the market, though the parts (or convincing and functional fakes of them) are available for less than half the price from Chinese sellers (ex on aliexpress). See the note suspicious parts below. With current of only 30mA maximum, but requiring only three external capacitors and no inductor and being much easier to design for.


## More things to know
* All switching power supplies can produce noise if not well designed (of both electrical and acoustic nature - the acoustic noise is generated by the fluctuating magnetic field of the inductors and from the pizeoelectric effect in ceramic capacitors as the electric field fluctuates, either way shaking the components and circuit board). The frequencies involved are usually high enough that they are not audible, some lower harmonics may be audible, and if there is a resonance in the system, they may be amplified. As you can imagine, if the converter is struggling to maintain it's target voltage, the changes in the electric and magnetic fields would be larger, making it more likely to emit detectable noise (of either type); this is not only a nuisance (either directly as acoustic noise, or by causing interferance or )
* Layout (hence, the size of the current loop) has a large impact on the noise generated by a switching converter. Appropriate sizing of the converter for the load and good circuit routing and layout are both major factors, but all sorts of weird things can have an effect, so the correct strategy is to keep the current loops as small as possible,
* You are better off starting with a higher voltage and bucking it down than starting with a lower voltage and boosting it up.
* The losses witin the converter is caused by both the overhead of the controller, and physical phenomona like parasitic resistance and the generally non-ideal beehavior which degrade performance. The first dominates at very low currents and the second at moderate to high current.
* **Don't design your own DC-DC converters** Can it be done? Of coures. But is it a good use of time? Almost never. Even experienced designers often often need several respins to get the device working to the required specification.
* "Uh so how do I get em?"
  * **You buy some of the abundant and inexpensive DC-DC converter modules from china**. Unlike individual chips, it is rare indeed to purchase an assembled board that doesn't function approximately as advertised. Yeah, the maximum current spec might a little like a fishing story ("I once caught a fish this big" *gesturing* "Yeah, I was there, I was holding the microscope"), the claimed efficiency a bit of a stretch, the usual Don't expect them to meet the specs entirely. but if you derate them a bit, they're pretty goodm . But if the company that had the boards made got back nothing but non-functional trash, discover they were all bad, and call out the suppliers of the junk, and refuse to pay or and take legal actions, which we can't really do half a world away.
  * The next step up is Pollolu - these guys charge about 3-4 times what the aliexpress vendors do, but they have a better reputation for quality.
  * Next step up is Adafruit, as you would probably guess. Lady Ada always has good product.
  * The top tier of DC-DC converter are the adorable self countained ones available from digikey. No, you didn't just accidentally swtich the prices to something with way less purchasing power, that's really what they cost.
  * Notice that $parkfun is not on the list. Their quality is not convincingly better than the aliexpress shit, at best on a par with pollollu, they have a history of misselling parts (like their "1 product number in their catalog to 2 product numbers, their choice" - one of them just barely worked at 3.3v on the gate (though it was out of spec) and the other didn't. Despite the fact that at the time, thhere was a TO220 MOSFET transistor which was spec'ed to be on with 3.3v on the gate. Vds(max) was the same, Rds(on) lower and it wasn't that expensive. That's not what Sparkfun chose to sell. They've also shipped my father parts with inaccurate documentation - quickly fixed by reading the relevant datasheets, their prices are higher than adafruit and they aren't really standouts at anything. Sorry - I just cant endorse a more expensive, inferior product because of it's brand name. Oh, and they're the ones who sold me the adapter with the counterfeit FTDI chip on it that got bricked during FTDIgate, leaving me to disassemble the lighting controller and replace the serial adapter by flashlight (cause without the serial adapter I couldn't upload the that would turn on the lights)

### Synchronous rectification
So remember those diodes in the topology? Those have a voltage drop that is significant. That's not desirable. Your finer modern converters make sure of a technology known as synchronous rectification. Nor only does the conroller switch the MOSFET as described, above, the diode is replaced by a MOSFET, allowing much lower voltage drops, and hence far higher efficiency. Wherever possiblem you should try to use converters with synchronous rectification. How do you find those? Unfortunately, for most affordable modules on marketplace websites - there's only one way to tell... Examine the pictures - what do you see:
* 3 capacitors and 1 central chip is a cap-swticher, and is only good for low current (though is is the simplest to design with and these you can design yourself assuming experience)
* 2+ capacitors, 1 central chip (possibly external MOSFETs when the power gets higher), 1 inductor (2 for buck/boost), and 1 or 2 fairly large diodes: These do not use synchronized conversion, and are cheap (or very large) - if it's neither, and it's got diodes visible on the PCB, you're getting ripped off.
* 2+ capacitors, 1 central chip (again possible external fets), 1 or 2 inductors as above... but no diodes. This is the signature of synchronously rectifier; try to use these. The difference in power consumption can be massive (I have seen improvements on the scale of 30%, moving from some beefy looking yet remarkably poor buck converters to some incredibly wimpy looking ones on a board less than half the size which absolutely beat the stuffing out of the old ones. The magnitude ofthe difference took me by surprise.

#### Approximate idealized losses
If we imagine all components are realistic but simple, that is, for current I, V<sub>SWdrop</sub> and V<sub>Ddrop</sub>, the voltage drops across the main switch or the diode or synchronous rectifier are approximated

V<sub>SWdrop</sub> = I<sub>in</sub> * R<sub>sw</sub>

and either

V<sub>Ddrop</sub> = C + I<sub>out</sub> * R<sub>diodedynamic</sub>

or

V<sub>Ddrop</sub> = I<sub>out</sub> * R<sub>swsync</sub>

and the inductor will always cost us

V<sub>Ldrop</sub> = I<sub>out</sub> * R<sub>L</sub>

So, if we notice that the syncronous rectifier simply has a different effective resistance, and C = 0, instead of 0.3-0.5V, for a buck converter, we get:

P<sub>waste</sub> = I<sub>in</sub><sup>2</sup> * R<sub>sw</sub> + C * (I<sub>out</sub> - I<sub>in</sub>) + (I<sub>out</sub> - I<sub>in</sub>)<sup>2</sup> * R<sub>eff, diode</sub>

While for a boost converter, it's just slighly different

P<sub>waste</sub> = I<sub>in</sub><sup>2</sup> * R<sub>sw</sub> + C * I<sub>out</sub> + I<sub>out</sub><sup>2</sup> * R<sub>eff, diode</sub>

Now at absolute best, I<sub>in</sub>/I<sub>out</sub> = V<sub>out</sub>/V<sub>in</sub> so

I<sub>in</sub> = V<sub>out</sub>/V<sub>in</sub> * I<sub>out</sub>

Let's call Ratio = V<sub>out</sub>/V<sub>in</sub>

P<sub>waste</sub> = (I<sub>out</sub> * Ratio)<sup>2</sup> * R<sub>sw</sub> + C * I<sub>out</sub> + (1-ratio)^2 * I<sub>out</sub><sup>2</sup> * R<sub>eff, diode</sub>

So let's consider a buck converter with a 1:4 in:out voltage ratio

P<sub>waste</sub> = 1/16 I^2 * R<sub>sw</sub> + C * 3/4 I + 9/16 I^2 * R<sub>eff, diode</sub> + I^2 * R<sub>L</sub>


P<sub>waste</sub> = I^2 * (R<sub>sw</sub> * 1/16 + R<sub>eff, diode</sub> * 9/16 + R<sub>L</sub>) + C * 3/4 I

So we have I^2 times the weighted sum of three resistances, where the weight of the main switch and the "diode" switch are Ratio^2 and (1-Ratio^2) and the weight of the inductor is 1.

If C is 0 as in a sync rectifier, then, it comes down to those three resistances, with the importance of the two switching elements being determined by the square of the voltage ratio; assuming the two fets are equal (they're not), it would in total be  I^2 * ((0.5 to 1) * (R + R<sub>L</sub>). And the inductor loss is always proportional to the square of the output current, with no dependance on

So in absolute terms, the resistance of the inductor matters most,

Then we add in that loss in the inductor.

## DC-DC converter modules
Since making ones own modules is difficult and likely to involve many revisions, and since prices of assembled modules are low, this is the preferred route (let DC-DC converter design experts deal with the details).
However, the quality differences between modules are profound. A listing of the most common ones is below. Except where noted, the queiscent current (drawn even at 0 load) is invariably larger than one would like it to be (hence, when using battery powered devices, you need to pay very close attention to the Iq vis-a-vis your power budget and requirements). This is the real point of this document, a guide to selecting good, cheap, DC-DC converters. So without further ado, here's a list of common and cheap DC-DC converters that can be had online

### Buck converters

#### Mini-360 2-stars.
Fixed 5v These are notoriously flaky, and are not recommended. They do not meet spec. Teardowns have been posted across the web where users claim to have found the design to be inconsistent with the datasheet, and speculate that this could be why, for example, they sometimes fail to start up under load.

#### Mini-560 4.5 stars (would by 5 if they didn't obfuscate the part number!)
Fixed 3.3/5/9/12V buck converter, Vin up to 20V, claimed current up to 5A (Optimistic, I reckon), but with sync rectification, it may not be far off. The chip has has the identifying marks ground off it, raising some questions about their provenance. Oscillations on startup may be significant, and use of zener on the input i particular can greatly help to miticate this. I have used well over a dozen of these for personal projects. The reduced heat generation is truly mindblowing, and after I saw the magnitudfe of the improvemen, I immediately began disassembling a large number of devices and swapping the buck converters. It was a large enough difference that it mattered not just in terms of saving power, and not just because the hot DC-DC converters raised fire hazard concerns - but also because I was having to throttle the total brightness of the string of LEDs I was driving with the old ones, as they'd overload the laptop power bricks I used for power, causing them to turn off, if the LEDS were on average of >95% duty cycle on all channels. With the new ones, it looked like I might even be able to add another section of lights (250 instead of 200)!

#### HW813 - 4.5 stars
Adjustable or fixed output configured by a pot, or cutting and bridging solder bridge jumpers. Synchronous rectifcation and an excellent module. I have used in excess of a dozen of these for personal projects

#### DD4012SA - 3.5 stars
Alas, this is not synchronos, efficiency suffers and the 1A maximum current is disappointing, It's high rating comes from another spec: Maximum input votage of a whopping 40V! (output voltage is factory set to 3V, 3V3, 3V7, 6V, 7V5, and 9V0 and 12V0)

#### LM2596 - 1 star
If I subscribed to the "if you don't have something nice to say, don't say it" school of thought with product reviews, there wouldn't be any text here. These modules advertise specs that exceed those of the components involved, tend to cook themselves under larger loads. You must have a means of limiting currect just a bit if a short is possible too - becasue a dead short on the output will crack open the main IC, producing what looks like a brief orange flame (a few ohms of resisance on the output is enough to prevent this.) These are adjustable, which is certainly nice. But wihin less than a month, on one project, accidental slipups resulting in shorts in the output burned out three of the bloody things. The efficiency is also poor nothing short of abysmal, the specs given for the assembled unit are inconsisent with those from the part datasheets. Lowest common demonminator, and should only be used in the least demanding applications.

### Boost converters

#### Brand/part number unknown, 0.9V to 3V3 input voltae to 3V3 or 5V0.
Green or blue PCB. Not syncrhonous rectification. Tests pending to see how good these *actually* are

#### Brand/part number unknown, 0.7V to 3V0 to 3V0, 3V3, or 5V0 volts.
Yellow PCB. Very low Iq. These may be sychronous. I have not tested these yet

#### ES01 4/5
6W maximum power, Vout = 5/8/9/12V, Vout > Vin > 2.5V. Voltage selectable with solder bridge jumpers.

#### MT3606 3/5
Adjustable voltage, Not sync. The good: wide input range, adjustable voltage. The bad: Poor efficiency, and very high Iq.

### Buck-boost

#### Brand/part number unknown 4/5 stars
2.5V to 15V to 5V @ 0.6A. Not the most inspiring converter, but they're fairly cheap and are buck-boost, which is rare.

### Dual rail

#### DA1712PA - 4/5 stars
This (as well as a few others is non-synchronous rectification converter which, it would appear, combines a boost converter with a charge pump, to generate a dual-rail output, ie, instead of +9V alone, both +9v and -9V). This sort of thing is useful mainly for powering analog devices which require a dual supply. Note that almost universally, dual rail DC-DC converters require the load on the positive rail to exceed the load on the positive rail.

#### The dirty +5 to +/-10 1-IC trick - the MAX232 - 2/5
Use a MAX232 (cheap ones can be had on aliexpress, likely clones,but they work). Intended to level shift TTL serial to RS232 voltage levels, this meant that The chip has to create +12 and -12V rails to convert TTL serial to RS232 voltage levels. This can be used to get low current (the meaning of low depending on the size of caps on the output and duration of the pulse) current dual rail supplies. The popular HC-SR04 ultrasonic ranging uses at least 1, and maybe two of these (with fets to put them in series when generating the ultrasonic pulse, thus giving them a much higher voltage to to drive the mini-ultrasonic speakers. These have all the same problems as cap swtichers, and are not as good at their job as something made as a dual rail converter, rather than a transciever that happens to have the on-chip charge pump pins accessible.

##### Summary (TLDR)
* Don't design your own DC-converter unless you think this document is a rag because you knew everything here already. Be prepared for in-depth study of relevant documentation and one or more respin!
* Use cap-switchers only if a small change in voltage is reuired, and the load current is very low (these you can typically be successful designing yourself without so much difficulty, at least).
* When curent is significant try to use synchronous rectification, The power savings is massive.
  * Not all converters say whether they are synchronous, but a picture is worth many words - look for the large diode, that looks fit to carry the whole load. If you see one, that's almost certainly not a synchronous converter. (the inverse is not true - the diode could be integrated into the main IC).
  * You will know quickly from it's efficiency (or lack thereof) whether it is once you start using it. The difference is not subtle.
* Chinese modules are probably your best bet for DC-DC converters unless you're an expert in that field of electrical engineering, or are such a baller tht you can buy self-contained modules fom western suppliers. However, especially if the module is not sync, you should plan to test it under real-world conditions.

## When should I skip a switching regulator and use a linear regulator?
* If the current is low enough that the linear regulator in a common package like SOT-223 will be able to handle it, and power use isn't a big deal - they are simple to use and design into your boards.
* If you are doing analog stuff, the noise from the switching regulator may cause problems. Using linear regulator can produce cleaner output. Say you had 20V coming in, but you only want 5v, and you need it clean. You might buck it down to 6-7V, and then pass it through a 5v LDO regulator, to get a more stable supply.
* If your input voltage is lower than the required voltage, you have no choice but to use a switching regulator.
  * Capswitchers are appropriate only for very low current. For example, an HV programmer for an AVR that used a MAX662A to generate the 12V.
* If the average current is very very low, such that it is on the same scale as the quiescent current of the regulator - good linear regulators have much lower internal power consumption than switchers.

## When should I not even think about a linear regulator
* Whenever the current is much over half an amp, you need a switching regulator - maybe 1A if the linear regulator has very low dropout and you're only dropping a fraction of a volt.
* When the input voltage is lower than the required output voltage - there's no way to increase the voltage without a switcher.
* When the input voltage is much higher than the output voltage. Convert the drop through the regulator at the maximum current into watts `(I * (Vin - Vout)) = Power (watts)`. Your typical SOT-223 regulator will struggle to dissipate more than around 1W. Just because a linear regulator might take up to 18v input, if you're getting 5v out of it, you will hit the thermal limits long before you get near any of the headline spec maximums for current - at just 75mA, the regulator will already be dissipating a watt!

## Can I ever avoid using a regulator altogether?
Absolutely - and when running on batteries, it's what you would always like to do.
You can just power it straight from the batteries. If you have 3 alkaline cells, that'll get you 4.5v nominal, 4.8-5.4 brand new. If you're using say, a Dx-series with no voltage dependance on it's speed grade, you could ride those batteries all the way down to 0.6V per cell. If you spend most of your time in power down or standby sleep, losing the quiescent current of a regulator is a Big Deal. This is also the recommended way to use a protected LiPo battery to power an AVR. The voltage will range between 4.2V fully charged and 3.7V for most of the discharge curve, which is within spec. Make sure the battery has overdischarge protection (UVLO - UnderVoltage Lock Out). And - again - as long as it's spending most of it's time sleeping, or it's current is small relative to the charging current of the battery during charging, you can just use a charging board to charge battery while it's still powering the chip. Otherwise, you need a slightly fancier battery management board that has a separate connection point for the load, so that the current taken by the load while charging won't confuse the charger. With NiMH batteries, which have around 1.25V nominal current, you'd probably want to use 4 of them.

## Fake parts
Buying parts direct from china (typically via AliExpress), one will find a varying "china discount" depending on the part. This effect is the smallest for semiconductions, and when there is an unusually large "china discount" (as there is on MAX662A chips) there's a significant chance that someone in the supply chain is a dishonest crook; The price of a MAX662A on AliExpress is a less than half what is is from Digikey. This is typically a red flag, though in that case at least, the suspiciously cheap MAX662A's appear to be either surplus stock (as the parts are seeing greatly decreased use) or they were cloned by a some asian semiconductor firm - but one who simply knocked off the design (or somehow obtained a copy of the masks) - "Maximum Price" Maxim's list prices for everything they make are eye-wateringly high, much like newer designs from TI, Analog Devices (and particularly LTC, which is also owned by Analog Devices now) - even when the device is quite simple. That leaves room (if the part is still widely used in china, provides a strong motivation for someone to make a working clone that costs less) for dishonest manufacturers to clone the functionality, undercut the manufacturer, and still pocket hefty profit. There are thus two very different classes of fake chips for sale by chinese vendors: Usually, for complicated ones like microcontrollers - ones where the cost of developing a clone would cost too much to make a functional counterfeit, and they just ship non-working trash to foreign customers who have little recourse. Meanwhile conceptually simple chips where less-than-ethical competitors can act as working "second source" providers, and sell a pin-and-spec compatible chip to supply domestic demand for less do exactly that. In this case the parts are usable, often with little sign of anyting not above board except for low price - and the manufacturer thus doesn't have to write a datasheet, much less one in english, while by claiming to be the original article, they can benefit from name recognition, saving on marketing costs. Other examples of the second class include the FT232RL clones - they act like FT232RLs, to the point where I'm sure they got their hands on the masks at some point - but they're not actually made by FTDI.

Normally western manufacturers' parts aren't much cheaper in china, while chinese semiconductors are straight up absent from western resellers), and when the parts in question are advanced parts, or parts that have a very complicated functionality, typically a sign of fraud, such as the remarking of parts like several the ATTinyCore users who bought "ATtiny85" parts from AliExpress encountered. They raised an issue about the wrong signature being read from them... Well, the sigature matched the capabilities of the part dead on - their supplier had simply bought a bunch of ATtiny13's, ground the markings off, and marked them as tiny85s. Other users of megaTinyCore reported 412's that thought they were 416s - but not properly functioning ones. Microchip hasn't told me whatever became of their investigation, but my theory is that a screwup was made at Microchip, the chips were recognized as bad, thrown away - but then pulled out of the waste stream by dishonest electronics sellers. Similar things have happened, historically, when particular parts got scarce for some reason (memorably, this happened to the m328p when it was fairly new, and at least two well known batches of fakes made the rounds - in one case, TQFP32 pakages with no die or internal connections to speak were manufactuered (indicating that the perpetrators had access to a full IC packaging facility), and in the other, a totally different chip was re-marked as a 328p. The frustrated buyer, a well-known hobby electronics retailer, posted die photos obtained of the chip with the help of some Atmel failure analysis folks, and posted it asking what the hell it was - and a flabbergasted representative of the original manufacturer contated them, asking in disbelief how they had obtained them, and explaining that they were an early  (engineering sample) version that was never shipped to customers. One assumes that more of those were produced than were needed, and had been sitting around somewhere, were eventually junked when it became obvious that they wouldn't need he used, and then fished out of the trash and re-marked as a chip that was in high demand).

Some other examples include SI23xx MOSFETS (Vishay makes the original. But a chinese company, which appears to have *similarly high end trenchfet technology* (whether licensed, stolen, or independently developed), Micro Commercial Corp *also* makes mosfets with the same part numbers and virtually identical specs. That's why you'll pay $0.25 cents in quantity for SI2302 fets on digikey (made by Vishay).... but around a penny each on aliexpress (for ones made by MCC. As I said though, MCC has a top of the line trenchfet process, and these FETs actually seem to work well enough; They also sell affordably priced large MOSFETs that are not direct copies of other companies products), consistent with the general trend that suspiciously cheap chinese parts which are simple are drop-in replacements and often actually work, while full on microcontroller that appear to be sold at TGTBT prices generally are NOT (note: Linear voltage regulators are another place you want to avoid cheaping out with chinese sources, particularly for AMS1117, which attempt to be have working/usable products, many of them have far worse performance at both steady state and in handling transients, and in some cases, the thermal shutdown is, ah, not self resetting, if you get my drift.

In general, substituting such suspect parts for known legit ones is often a tough call - the savings can be hard to resist, but testing should **always** be conducted to ensure that the parts behave as promsed. The more experience and the more testing, the better off youll be.
