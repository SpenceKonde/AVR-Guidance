# Stacking Pin Header Remystified
Stacking pin heaer, quite simply, is a type of 0.1" pin header which has female sockets at one hand, and long male pins - typically on the order of 0.4" to -0.6" - on the other. Hence, if soldered into a PCB, it can be, as the name implies, stacked. Indeed one can imagine a stack consisting of an arbitrary mumber of identical PCBs, with identical pieces of stacking pin headerin them, stacked skywards. The most common example is the classic Arduino shield, where multiple shields might be stacked atop eachother on top of an Arduino Uno or similar. This makes it very useful for making both flexible development and accessory boards, and for connecting multiple devices to a single microcontroller - as is done in the classic shield. Far more common in 6p, 8p and 10p sizes due to their use in Arduino shields. While one might expect 2x3 to also be common, surprisingly, it is significantly less common than the other two.

So why do these get their own page? Well, like many things in electronics (and elsewhere) - it's more complicated than it seems. In this case, it all boils down to a single issue...

## Quality Varies - a lot!
It is at least soemwhat common knowledge that there are more than one kind of stacking pin header, and that they are not all created equal. What is not common knowledge is just how profound the difference in quality is. This leads, for example to some people declaring it to be useless garbage and advising others to not touch the stuff with a ten foot pole (I don't know how we're supposed to throw it out then), and others telling them that they must be incompetent because they usually use stacking pin header and never have any problems (with the pin header at least, I don't know anyone who doesn't have problems with some of their projects sometimes).

It is entirely possible that both people are speaking in entirely good faith, despite saying different things, because the difference between good and bad stacking pin header is as great as the difference between good and bad "183C lead free" solder paste (see [my solder and soldering guide](Solder.md), but suffice to say that the category includes far and away the worst solder paste I've used as well as 2 of the top 3 solder pastes in the 140-200C liquidus ROHS bracket, of which I personally tested over a dozen, at least one of which thus far has been disqualified for containing lead, making it's failure to take the #1 spot even more humiliating).

There are two sides to this problem of quality. The first one being the sheer magnitude of the difference. The good ones are probably better than normal female pin header, while the bad ones are not only not usable on the female side, the male pins don't make reliable contact when plugged into good female header either. Bad stacking pin header is incredibly bad. The other side to the problem is that, while viewed in person, someone who has seen both kinds will typically recognize the good ones instantly, the mediocre and bad ones are difficult to tell apart without a good dial caliper (accurate to 0.01mm), and almost all of them look identical in an ebay or aliexpress listing. 

Stacking pin heasder can be best identified by measuring the dimensions of the male pin end - other methods of visual identification are prone to error until one has examined to two kinds next to eachother. Cheap or low quality header has flat pins, made by stamping thin, springy metal: the width is much greater than the thickness. The highest quality header has square pins. Additionally, there are sometimes more dramatic signs of poor quality: If you get your bag of stacking pin header, and some of the pins have fallen out, you didn't get the good kind. A first estimate of quality, lacking any tools, can be made with two pieces of the stacking pin header, and a normal male and female header. Ideally, in all combinations, the pins could be slid in and out with gentle force, like normal male and female pin header, and such action will be smooth. Low quality stacking pin header can be felt scraping against the mating connector, and if the edge of the pin is examined, you can typically see that the edges of the pins are not smooth.

Only the highest quality stacking pin header will meet the above standard. Typically lower quality stacking pin header will mate well enough with standard header, but trying to use multiple layers will be less successful.

Pin header of all types should offer some resistance while being inserted and removed; if it does not, that suggests that it may not be making contact at all, or if it is, it is just resting against the other conductor, and pressure in some direction would cause them to no longer make contact. When both the male and female sides are made of foil-thin metal, this is a common state of affairs (and why sandwiching a single stacking pin header between two normal ones will often work). 

Pin header (of any typer) also should never be used if it needs to be forced into the matching pin header. Indicative of pinheader designed to meet a slighly different parameters, or an absense of any quality control, forcing such header into use will rapidly damage both sides (leading to the previous situation)

## Types seen in the wild
Names are arbitrary, as there is only one connector with any part number associated with it at the consumer level. They are nearly impossible to differentiate from pictures (and that's all you usually get to judge on with these. All you can do is buy a sample, see if they're good when they arrive, and buy more if they are.

### The Good Ones - 0.62mm square pins
6p, 8p, and 10p with 15mm pin length [this aliexpress listing is an example](https://www.aliexpress.com/item/32991245965.html) (no connection to seller, other than that I buy lots of crap from them). The same seller also sells inferior ones.

8p in 11mm - seen bundled with some RobotDyn products, such as their ESP8266 USB-serial interface board (a D1-mini style board, but with no ESP-8266 module, so that you can install your preferred module). Has not been spotted available separately - unfortunately, as the 11mm length is more desirable, generally speaking...

Once you've seen what they look like, you will no longer need dial calipers - the appearance of pin tells you what you have. 

### Class 1.5
These are very recognizable. Search for PC104 header, and you'll find some listings where there's a second piece of plastic at the base of the first one. This is what I term "Class 1.5". The only reason they come after the Good Ones (Class 1) is that that extra spacer tends to get in teh way What is really appealing about these is that they come in a 1x40 (and 2x40) size, and both of these are relatively inexpensive?. Who wants a 40-pin long stacking connector?! I don't, You probably don't, if you think you do, there's a good chance that you're solving a problem the wrong way. doing something that's not such a great idea. Anyway, the value is that these have good pins, at a low price per pin. They're your raw material when you need to make stacking header of different parameters (colored housing, specific length that isn't 6, 8 or 10).

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

These are the most common ones available. Unfortunately, they are utter garbage. They should never be relied upon to mate with other class 2 or 3 stacking pin header. This is particularly disappointing, as they are available in the widest variety of sizes, too. Worse still, the difference in dimensions is incredibly small.  

## What, exactly is wrong with the crap header? 
As far as I can tell there are two issues. The pin is thinner, so when two stackable headers are stacked they can miss eachother. The second can be felt as you insert the header (either end) - the edge is rough, leading to the electrical contacts wearing out quickly

## Identifying good ones in listings
It is virtually impossible to tell the difference between good and bad single row header until you open the package shout profanity and throw them into the garbage can. Actually if you're selling products that include stacking header, make it a public trashcan a ways away, you don't want anyone to know that you had any of the bad ones or they will assume you used them in your products, and if they have any sense, will think you're selling products with trash header and avoid you like the plague. That is my recommended practice if you locate vendors selling products made with crap stacking pin header. Howe

### If you dont't mind that extra spacer between the normal female portion, and the pin
The PC104 ("Class 1.5") is entirely suitabe, and can be easilty spotted in listing photos. If you need or want the edge to be clean, like all female pin header, you need to first cut it in the middle of the first unwanted pin, and then clean up the edge (a razor blade works best for this, and will get a smoother edge than wire cutters. Unfortunately, it is also excellent for cutting your fingers - be sure to rinse the blood off the set of terminals you were working on too - it isn't very visible against black plastic, but it if itgets into the holes, it can interfere with the electrical contact.

### Double Row header
Double row header, as well as Class 1.5 (PC104) is far easier good specimes of: good double-row stacking pin header has a block of plastic that looks like the plastic around the pins in ordinary dual row pin header between the base of the female receptacle and male pins. I have not captured any double row stacking pin header better than Class 3 that does not have this.

2x20 is by far the most common size - Why? because it's what the Raspberry Pi uses for it's GPIO header!

### Colored stacking pin header is always the worst of the worst
Unless you make it yourself, for whatever reason, no manufacturers make quality colored stacking pin header. 

## Making custom header pieces
This doesn't scale well unless you have a flunkie who will do piece work for you., but if you want a small quantity of, say, colored stacking pin header, you can do it.

1. First you need some good pins. Start with a 1x40 strip of the PC104 type. Separate the plastic parts, maybe an eighth of an inch of daylight between them, amd then go down the whole 40-pin connector with pliers, yanking each pin up an 8th of an inch (at which point the wide part inside hits the other plastic thing and won't come further - but now it's disloged from the larger piece) When done, the other part will fall off. Now, remove the pins from the plastic spacer (whcih is probably all mangled and ugly by now, throw it out). Now you have a pile of long, good pins. 
2. Now you need will also need a housing to put them into: start with some normal female header of the correct parameters. Yank the pins out one at a time with pliers. 3. Replace them with the new pins which you will find fit in perfectly. Press them in place. To make wierd 90-degree angeled pin header, insert the pins into two pieces of small prototyping board, and bend as desired. (using two pieces of protoboard ensures a gentle bend.
4. Finally, you need that plastic spacer that goes on the bottom. For that - you guessed it - male header and more work with the pliers. At this step, you can easilly snap the plastic. The side with the indentation is *supposed* to face the board - it often doesn't on colored pin header though. 

Like I said, this doesn't scale well, but it's definitely not as slow as it sounds. It goes quick enough that it's not time prohibitive for small quantities, and a minion who worked reasonably quickly for $15/hr wouldn't break the bank, unless you needed hundreds of them 

### Advanced variations
Combining stacking pin header, 90 degree bends, and small pieces of protoboard can yield bizzare stacking assemblies can be created. Some of the demonstrations of the technique I have made are clearly useless - but concept has merit. The real problem is that the 0.1" pin header ecosystem is sort of awash with crap - some female header is trash and has a short lifespan or fails after only a few connection cycles, dupont line is disposable (and has a disturbingly short lifespan) unless made with expensive (30 cents per terminal) terminals with the original DuPont design, and the commercial dupont line is typically made with undersized wires (AWG 2-4 smaller than marked) with insufficient force (likely to reduce wear on the crimping machiene, or the wire is so thin that they have it cranked to the max and it's still not able to apply enough force) used to crimp (substandard connectors based on the Harwin M20, only modified to use slightly less, thinner metal. They rely on the same mechanism for both retention of the connector and the electrical connection, which is how they fit into 0.1x0.1 grids, but brass isn't very springy and tends to deform, and it's exposed to not only the normal stresses of connecting and disconnecting, but to the wire's every yank and pull).In say a JST-XH connector, force is transferrred from the pin to the housing via one latch, separate from the spring that makes the electrical contact.
