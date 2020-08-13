# General Hardware Design (not AVR-specific)
This document/directory contains some not-particularly-organized notes, tips, and tricks I've picked up which are likely particularly relevant to those making boards for personal projects and/or for home assembly. 

## Reducing part count
Are you placing parts on the board with tweezers? Yeah, I hate that too. Obviously, anything we can do to reduce part could is a boon as long as it doesn't result in yield problems.

### Resistor networks are your friends
Are you placing more than one of any value of resistor near eachother? If you aren't using automated assembly, stop doing that. You can get resistor networks with 2, 4, or 8 resistors in one chip-resistor package! The sweet spot seems to be 4 in a pakage. The 8-resistor ones are limited in selection and much more expensiuve, while the 2-resistor ones are, for reasons I never really understood, often far less cooperative when soldering them down. I would suggest 1206 4-resistor ones with concave terminations (the convex version is cheaper, but harder to solder consistently). 

### Replace multiple indicator LEDs with RGB or bicolor LEDs
Consider replacing individual discrete LEDs with a smaller number of RGB LEDs or "bicolior" LEDs - these are packages consisting of two LEDs next to eachother in one package. 
A few tips
* 5050 RGB LEDs make ugly indicators. 1206 or 606 packages look better in this application.
* With 1206 and smaller packages, you can't see the individual LEDs clearly as points of light, even at fairly close range. With 5050s, without a diffuser, you can see the individual LEDs from halfway across the room. That's a big part of why I say the 5050's look lousy. 
* Aside from above caveats, WS2812 LEDs can be a real gift here. I have seen a number of commercial products that used a WS2812B or string of a few of them as indicators in order to let them use a smaller, cheaper microcontroller with fewer pins. One can argue how much of a win that design decision was - with the current AVR product line, it wouldn't be, but back then, they were in a bit of a tight spot, as they needed two hardware serial ports, but hardly any other resources - so their options were a $2 ATtiny1634, or a $8-or-whatever ATmega1284P.Thankfully, Microchip has brought the costs of the AVR line way down since they took over, so it's not quite as painful to reach for a megaAVR or DA-series, instead of a tinyAVR. 
* 606 package are a pain to work with by hand - but you can get bicolor and RGB LEDs in a variant of 1206, which are much easier to handle. The terminals on either end are split down the middle, so 4 terminations per part means enough for an RGB or 2 individual LEDs.
* Remember to check the brightness of the individual colors in these multi-emitter parts! For example, one manufacturer makes a Red-Green where the green is many times brighter than the red - you can't even tell if the red is on when the green is, unless you give the green a much larger ballast resistor (but you're trying to avoid that, because you don't want to have to place different values of resistors). But that same company makes a Red-Yellowgreemn version too, where they're about the same brightness.
