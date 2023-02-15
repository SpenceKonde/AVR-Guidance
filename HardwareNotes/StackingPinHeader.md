# Stacking Pin Header Remystified
Stacking pin heaer, quite simply, is a type of 0.1" pin header which has female sockets at one hand, and long male pins - typically on the order of 0.4" to -0.6" - on the other. Hence, if soldered into a PCB, it can be, as the name implies, stacked. Indeed one can imagine a stack consisting of an arbitrary number of identical PCBs, with identical pieces of stacking pin header in them, stacked skywards. The most common example is the classic Arduino shield, where multiple shields might be stacked atop eachother on top of an Arduino Uno or similar. This makes it very useful for making both flexible development and accessory boards, and for connecting multiple devices to a single microcontroller - as is done in the classic shield. Far more common in 6p, 8p and 10p sizes due to their use in Arduino shields. While one might expect 2x3 to also be common, surprisingly, it is significantly less common than the other two.

So why do these get their own page? Well, like many things in electronics (and elsewhere) - it's more complicated than it seems. In this case, it all boils down to a single issue (would that more problems lent themselves to such straightforward analysis)... and that problem is well... Essentially the whole point of a stacking pin header is that an electrical connection can be made to the thing stacked on top, the thing stacked below, and the thing in the middle soldered to the pins. That requires thoughtful design and careful execution. The result is that *Quality varies..... a lot*. 

## Quality Varies
It is at least soemwhat common knowledge that there are more than one kind of stacking pin header (though many may only be familiar with a single kind; since the vast majority is the lowest tier stuff described here, I can't blame someone if they tried it once, were horrified, and vowed never to use stacking header again). Even if you know that there are different kinds, you probably don't know just how large the magnitude of the difference is. 

This leads, for example to some people declaring it to be useless garbage and advising others to not touch the stuff with a ten foot pole (a constraint which makes it difficult to get into a trashcan), and others telling them that they must be incompetents or liars, because stacking pin header works great for them every time!

It is entirely possible (most likely in fact) that both people are speaking in entirely good faith (though, this being the internet, likely not kindly), despite saying opposite thigns things, because the difference between good and bad stacking pin header is just that big. Almost nothing sold with the same name and keywords is as profoundly different than grades of stacking pin header. 

There are two sides to this problem of quality. In both cases the magnitude of the differences are profound. 
1. First and often more important is the female side of the header. You can't see the quality of this part without disassembling it - but you can totally feel it as you plug something into it. If you haven't encountered crap pin header (normal, not stacking), you haven't been buying cheap Chinese pin header very much. Bad female pin header absolutely exists. You discover that it's bad when you discover that it's not actually making contact, or is doing so only intermittantly. A very frustrating debugging proposition, to be sure. Oddly, while the worst stacking pin header is worse than even really bad normal female headers, the best stacking pin headers are actually *better* than even fairly good female pin header (not that you'll notice pin header quality unless you're either specifically looking for it, or it's terrible and you can't ignore the problem).
2. Secondly, and unlike normal pin header, the male side can also vary greaty in quality; the "worst" male pin header you'll find is... just made with cheesy plastic that doesn't hold up to soldering temperatures, or that can't be cut cleanly because it's too brittle. The key difference is that standard pin header has *square* pins. It's very hard for it to not make a decent connection with either female header or female DuPont connectors. Most stacking pin header is instead made more like female header with the pin extended much further, that is, each contact is a little piece of stamped metal shaped like a tall, skinny "Y".

It's that manner of construction that leads to both sides of the problem. The cheaper the manufacturer is, the thinner the metal they'll stamp it from and the less smooth the edges will be. Ragged edges scrape away at the mating terminal, and that's what you don't want to feel when you connect pin header. The bad ones also have lightly shorter tines, which grip the pin less tightly and only make contact at all at their very tip. Similarly, the male part of the pin is narrower, not just thinner. Further, you may think it's gold plated - nope. While good non-stacking pin header is, I've never seen stacking pin header that wasn't golden color all the way through (ie, it's unplated brass or bronze). The line between brass and bronze is blurred - Brass contains mostly copper and some zinc and often tin as well, bronze contains mostly copper, some tin and often zinc as well
And they are both often alloyed in trace amounts with group V elements (particularly phosphorus and arsenic), or with other metals at concentrations as high as several percent to improve various properties (phosphorus as noted makes it springier, lead makes it machine better, and so on). Neither has the corrosion resistance of gold plating (not that it would last long on poor quality contacts, thanks to the rough edges). Gold flash is actually quite rare on any pin header - most female header is brass/bronze (and judging by how easily it deforms, i'd guess it's rarely if ever a proper phosphor bronze). Male pin header is commonly tin plated steel or brass (Western manufacturers do sometimes plate with gold, but you don't want to know what they charge) - brass and steel is the norm. 

These two problems are compounded when stacking header is plugged into itself. Unless it is the best type, since both conductors are thin in one dimension, there is a significant chance that they won't line up well, and both the male pin is narrower and the gap on the female end wider, as well, AND they have rough surfaces that wear faster. Plus the greater length serves as a lever, subjecting it to more mechanical stress. Oh, and the good ones have a retaining groove that helps hold them in place, the cheap ones don't/

## Telling them apart
Despite this huge difference in performance, it is very easy for even an observant individual to not realize when they're working with junk. Biewed in person, someone who has seen both kinds will typically recognize the good ones instantly, but anyone else, or different grades of inferior stacking header are difficult to tell apart without a good dial caliper (accurate to 0.01mm). Yet in photos? they look identical. 

Stacking pin heasder can be best identified by measuring the dimensions of the male pin end - other methods of visual identification are prone to error until one has examined to two kinds next to eachother. Cheap or low quality header has flat pins, made by stamping thin, springy metal: the width is much greater than the thickness. The highest quality header has square pins. Additionally, there are sometimes more dramatic signs of poor quality: If you get your bag of stacking pin header, and some of the pins have fallen out, you didn't get the good kind. A first estimate of quality, lacking any tools, can be made with two pieces of the stacking pin header, and a normal male and female header. Ideally, in all combinations, the pins could be slid in and out with gentle force, like normal male and female pin header, and such action will be smooth. Low quality stacking pin header can be felt scraping against the mating connector, and if the edge of the pin is examined, you can typically see that the edges of the pins are not smooth.

Only the highest quality stacking pin header will meet the above standard. Typically lower quality stacking pin header will mate well enough with standard header, but trying to use multiple layers will be less successful.

Pin header of all types should offer some resistance while being inserted and removed; if it does not, that suggests that it may not be making contact at all, or if it is, it is just resting against the other conductor, and pressure in some direction would cause them to no longer make contact. When both the male and female sides are made of foil-thin metal, this is a common state of affairs (and why sandwiching a single stacking pin header between two normal ones will often work). 

Pin header (of any typer) also should never be used if it needs to be forced into the matching pin header. Indicative of pinheader designed to meet a slighly different parameters, or an absense of any quality control, forcing such header into use will rapidly damage both sides (leading to the previous situation)

## Types seen in the wild
Names are arbitrary, as there is only one connector with any part number associated with it at the consumer level. They are nearly impossible to differentiate from pictures (and that's all you usually get to judge on with these. All you can do is buy a sample, see if they're good when they arrive, and buy more if they are.

### The Good Ones - 0.62mm square pins
Unsurprisingly, this is the same dimension as normal male pin header! You can get 6p, 8p, and 10p with 15mm pin length [this aliexpress listing is an example](https://www.aliexpress.com/item/32991245965.html) (no connection to seller, other than that I buy lots of crap from them). The same seller also sells inferior ones, though so watch out. 

8p in 11mm - seen bundled with some RobotDyn products, such as their ESP8266 USB-serial interface board (a D1-mini style board, but with no ESP-8266 module, so that you can install your preferred module). Has not been spotted available separately - unfortunately, as the 11mm length is more desirable, generally speaking...

Once you've seen what they look like, you will no longer need dial calipers - the appearance of pin tells you what you have before you've even taken them out of the bag - smooth shiny edges are good, and 

### Class 1
These are very recognizable. They are not quite as good as The Good Ones - but they're not bad! Search for PC104 header, and you'll find some listings where there's a second piece of plastic at the base of the first one (so it looks like afemale header with the plastic spacer from a male header. This is what I term "class 1". The biggest problem is that they don't come in the sizes you want, and are scarser and more expensive. However, the cost can be brought down by buying 2x40 or 1x40 pieces of it (which are available!). Both are are relatively inexpensive. Who wants a 40-pin long stacking connector though?! I don't, You probably don't either. But with patience you can turn small quantities of this into pin header of any size, even colored (see below)

### Class 2
0.37-0.40 x 0.57-0.60mm pins

6p, 8p, 10p, 15p in 15mm length

Available as part of a surprisingly cheap kit containing dozens of these, in one of the cheapest, crappiest plastic organizer boxes ever seen. These are the best of the rest - but that's a very low bar. While they don't slide smoothly, and their action hardly inspires confidence, they are generally good enough to interface with eachother. I would be concerned if they were frequently being connected and disconnected, and would always test-fit a pair of them that were planned for use together prior to soldering. Occasionally one encounters one with a less satisfying female side. With the discovery of the "class 1.5" I can no longer justify use of these for any purpose

### Class 2a
0.37-0.40 x 0.70-0.75mm pins

Only seen in 10p in the wild - source unknown.

Despite the strange measurement on the width of the pins, these are comparable to the other Class 2 stacking pin header

### Class 3
0.37-0.40 x 0.60-0.65

2p, 3p, 4p, 6p, 8p, 10p, 2x3p in 11mm length

6p, 8p, and 10p in 11mm length, red, yellow and blue plastic

These are the most common ones available. Unfortunately, they are utter garbage. They should never be relied upon to mate with other class 2 or 3 stacking pin header. This is particularly disappointing, as they are available in the widest variety of sizes, too. Worse still, the difference in dimensions is incredibly small making identification very hard. 

## Identifying good ones in listings
It is virtually impossible to tell the difference between good and bad single row header until you open the package and shout profanity and either put them triumphantly into the stacking pin header drawer, or angrilly fling them into the trash. Don't do the latter though - get your pair of pliers first. Yank the pins out the bottom of each plastic part - especially if they're the colored ones. Keep the plastic and discard the bad contacts (or if you sell scrap brass, they go in the brass bin). 

### If you dont't mind that extra spacer between the normal female portion, and the pin
The PC104 ("Class 1") is entirely suitabe, and can be easilty spotted in listing photos. If you need or want the edge to be clean, like all female pin header, you need to first cut it in the middle of the first unwanted pin, and then clean up the edge (a razor blade works best for this, and will get a smoother edge than wire cutters. Unfortunately, it is also excellent for cutting your fingers - be sure to rinse the blood off the set of terminals you were working on too - it isn't very visible against black plastic, but it if itgets into the holes, it can interfere with the electrical contact. 

A better method is described below


### Double Row header
Double row stacking header is rarer, but almost all of it is Class 1 (PC104). 

2x20 is by far the most common size - Why? because it's what the Raspberry Pi uses for it's GPIO header!

### Commercial colored stacking pin header is always the worst of the worst
Never buy it. If you are given it for free, I guess take it, but rip the pins out when you get it home and keep only the plastic part. The pins in colored stacking pin header are class 3 on a good day

## Making custom header pieces

There is a better way to get colored stacking pin header, at least in small quanties. Get some 1x40 or 2x40 class 1 stacking header. Also buy some female header of the lengths you want, and some male header. You can get colored header for this - yellow- green, red, blue and white can be had in addition to black. With a thin knife or razor, insert the blade between the two pieces of plastic and twist just enough that you can the a small flat screwdriver between them. Work your way along the header until you've gotten the two plastic parts about 1/10th inch apart. Now, get your pliers and pull the male end of each pin (one at a time - but the whole 40 pins are done in a minute or so); each should move about a tenth of an inch. By the end, the pins and male style spacer will come off entirely. Now you have a male plastic spacer with the good pins in it. press the male end straight down onto a table near the edge, and you can push up 8-10 pins at a time. Those can then be pulled out by hand. Thisprocess kinda mangles the plastic parts, so throw those out. 

Then, use pliers to yank the pins out of a male and a female normal header. Female header should be the desired length if possible, male header is cutable so it doesn't matter so much. Do the same to male header. Discard these pins (if recycling the brass for scrap, remember to use a magnet to remove the steel ones. If you used angled female pin header, save the angled pins: you can make colored female angled header the same way (male angled header is available in all six colors). Slide the stacking pins into the holes in the male plastic piece after cutting it to the required length(s), all the way in (remember, the groove goes towards the PCB, away from the female header. *Some colored pin header has this backwards*. If the spacer is upside down, it is possible to form bridges under the pin header (the narrow gap reqires very little solder to fill, and some solder will wick up, which is no fun at all. Then just put the female end into the female header, press the male pins straight down to make sure they're in all the way. Repeat until you have the connectors you need (and a pile of dead pins).  The same technique can be used to make double row, stacking pin header too. While small batches aren't too time consuming, it does not scale well at all, unless you have a flunkie who you can put to work disasembling header. 

### Advanced variations
Combining stacking pin header, 90 degree bends, and small pieces of protoboard and breakout board can yield bizzare stacking assemblies. While some of the demonstrations of the technique I have made are clearly useless - others aren't, and I do belive the concept has merit concept has merit. The real problem is that the 0.1" pin header ecosystem is sort of awash with crap - some female header is trash and has a short lifespan or fails after only a few connection cycles, dupont line is disposable (and has a disturbingly short lifespan) unless made with expensive (30 cents per terminal) terminals with the original DuPont design, and the commercial dupont line is typically made with undersized wires (AWG 4 smaller than marked is typical) crimped with insufficient force (likely to reduce wear on the crimping machiene, or the wire is so thin that they have it cranked to the max and it's still not able to apply enough force), onto terminals that are knockoffs of the knockoff of the harwin M20, which is a knockoff of the real DuPont terminals. (substandard connectors rely on the same mechanism for both retention of the connector and the electrical connection, which is how they fit into 0.1x0.1 grids, but brass isn't very springy and tends to deform, and it's exposed to not only the normal stresses of connecting and disconnecting, but to the wire's every yank and pull. Real dupont terminals are now made by Amphenol (DuPont spun out it's connector division as Berg to focus on their core business of manufacturing polluted groundwater, and then Berg was bought by Amphenol). Note the contrast to a JST-XH connector, where the plastic housing bears much of the force - at the cost of being wider than 0.1".
