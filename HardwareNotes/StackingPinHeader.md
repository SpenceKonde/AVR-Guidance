# Stacking Pin Header Remystified
Stacking pin heaer, quite simply, is a type of 0.1" pin header which has female sockets at one hand, and long male pins - typically on the order of 0.4" to -0.6" - on the other. Hence, if soldered into a PCB, it can be, as the name implies, stacked. Indeed one can imagine a stack consisting of an arbitrary mumber of identical PCBs, with identical pieces of stacking pin headerin them, stacked skywards. The most common example is the classic Arduino shield, where multiple shields might be stacked atop eachother on top of an Arduino Uno or similar. This makes it very useful for making both flexible development and accessory boards, and for connecting multiple devices to a single microcontroller - as is done in the classic shield. Far more common in 6p, 8p and 10p sizes due to their use in Arduino shields. While one might expect 2x3 to also be common, surprisingly, it is significantly less common than the other two.

So why do these get their own page? Well, like many things in electronics (and elsewhere) - it's more complicated than it seems.

## Quality Varies - a lot!
Rare indeed is the member of the Arduino community who is not aware that more than one type of stacking pin header exists, some of which is much better than others. The differences in quality are very dramatic in the case of stacking pin header - far more so than for most electronic components. Stacking pin heasder can be identified by measuring the dimensions of the male pin end - other methods of visual identification are prone to error. cheap or low quality header has flat pins, made by stamping thin, springy metal, whereas the highest quality header has square pins. Additionally, there are sometime more dramatic signs of poor qualitym, such as pins simply falling out. A first estimate of quality can be made with two pieces of the stacking pin header, and a normal male and female header. Ideally, in all combinations, the pins could be slid in and out with gentle force, like normal male and female pin header. You can typically see that the edges of the pins are not smooth.

In practice, only the highest quality stacking pin header will meet that standard. Typically lower quality stacking pin header will mate well with standard header, but not with other similar stacking pin header. Pin header should offer some resistance while being inserted and removed; if it does not, that indicates that it may not be making contact - or if it is, it is just resting against the other conductor, and pressure in some direction would cause them to no longer make contact. When both the male and female sides are made of foil-thin metal, this is a common state of affairs (and why sandwiching a single stacking pin header between two normal ones will often work). Pin header also should not be used if it needs to be forced into the matching pin header - this will rapidly damage both sides (leading to the previous situation) - though if it is only being connected once and left that way, it may be acceptable.

## Types seen in the wild
Names are arbitrary. They are nearly impossible to differentiate from pictures (and that's all you usually get to judge on with these. All you can do is buy a sample, see if they're good when they arrive, and buy more if they are.

### The Good Ones - 0.62mm square pins
6p, 8p, and 10p with 15mm pin length https://www.aliexpress.com/item/32991245965.html (no connection to seller, other than that I buy lots of crap from them)

8p in 11mm - seen bundled with some RobotDyn products, such as their ESP8266 USB-serial interface board (a D1-mini style board, but with no ESP-8266 module, so that you can install your preferred module). Has not been spotted available separately - unfortunately, as the 11mm length is more desirable, generally speaking...

Beautiful, smooth square pins. When you open the package, you don't even need to get out the calipers to know if you got The Good Ones. Pins are typically straight on arrival, do not bend easily, and look and feel smooth. These are the legendary good stacking pin header. Can be stacked with themselves without conern - I would have more faith in these than bargain basement normal pin headers!

### Class 1.5
These are very recognizable. Search for PC104 header, and you'll find some listings where there's a second piiece of plastic at the base of the first one. These actually work pretty well! (don't bother with the kind that don't have that). What is really appealing about these is that they're very inexpenmsive in 1x40 pin size. Who wants a 40-pin long stacking connector?! You probably don't, if you do, you're probably doing something that's not such a great idea. Anyway, the value is that these have good pins, at a low price per pin. They're your raw material when you need to make stacking header of different parameters (colored housing, specific length that isn't 6, 8 or 10).

### Class 2
0.37-0.40 x 0.57-0.60mm pins

6p, 8p, 10p, 15p in 15mm length

Available as part of a surprisingly cheap kit containing dozens of these, in one of the cheapest, crappiest plastic organizer boxes ever seen. These are the best of the rest - and that's a very low bar. While they don't slide smoothly, and their action hardly inspires confidence, they are generally good enough to interface with eachother. I would be concerned if they were frequently being connected and disconnected, and would always test-fit a pair of them that were planned for use together prior to soldering. Occasionally one encounters one with a less satisfying female side. With the discovery of the "class 1.5" I can no longer justify use of these for any purpose

### Class 2a
0.37-0.40 x 0.70-0.75mm pins

Only seen in 10p in the wild - source unknown.

Despite the strange measurement on the width of the pins, these are comparable to the other Class 2 stacking pin header

### Class 3
0.37-0.40 x 0.60-0.65

2p, 3p, 4p, 6p, 8p, 10p, 2x3p in 11mm length

6p, 8p, and 10p in 11mm length, red, yellow and blue plastic

These are the most common ones available. Unfortunately, they are pretty shoddy. They should never be relied upon to mate with other class 2 or 3 stacking pin header. This is particularly disappointing, as they are available in the widest variety of sizes, too.

## Double Row header
Double row header is far easier to spot The Good Ones: good double-row stacking pin header has a block of plastic that looks like the plastic around the pins in ordinary dual row pin header between the base of the female receptacle and male pins. I have not captured any double row stacking pin header better than Class 3 that does not have this.

2x20 is by far the most common size - Why? because it's what the Raspberry Pi uses for it's GPIO header!


## Making custom header pieces
This doesn't scale well, but if you want a small quantity of, say, colored stacking pin header, you can do it.
First you need some good pins. Start with a 1x40 strip of the PC104 type. Separate the plastic parts, maybe asn eighth of an inch of daylight between them, amd then go down the whole 40 connector with pliers, yanking each pin up an 8th of ann inch (at which point the wide part inside hits the other plastic thing and won't come further - but now it's disloged from the larger piece) When done, the other part will fall off. Now, remove the pins from the plastic spacer (whcih is probably all mangled and ugly by now, throw it out)
You will also need a housing to put them into: start with some normal female header of the correct parameters. Yank the pins out one at a time with pliers. Replace them with the new pins which you will find fit in perfectly.
Finally, you need that plastic spacer that goes on the bottom. For that - you guessed it - male header and more work with the pliers.
Like I said, this doesn't scale well, but it goes quick enough that it's not prohibitive for making one or two of something.
