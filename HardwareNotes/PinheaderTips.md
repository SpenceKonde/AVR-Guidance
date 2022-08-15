# Pinheader for pinheads - tips and tricks with pin header
This does not cover solder-related things in general
[The soldering "guide", which is more like an essay that would be rejected by any educator for excessive length](Solder.md)

## Basics
Soldering pin header is the easiest PCB soldering task around, and is often the first thing people solder onto a PCB, or the first thing they solder that isn't a simple through-hole part on prototyping board. It is thus often the point where people realize they're doing something wrong. Make sure you have a temperature controlled iron.

Knife-type tips are recommended for fastest soldering.

Leaded solder is recommended for easiest soldering, if available, but pin header is not a demanding task, and soldering it with unleaded solder is not particularly hard with a decent iron.

Leave the iron on the pin longer than usual if you are for some reason using solder paste (for example because the solder you need is not available as wire - out of over a dozen suitable solder alloys only 3 are available as wire (assuming all the >05% Sn 0.7% Cu Ag alloys, the 60/40, 63/37, and slightly silver-bearing versions of those leaded alloys, and the SnBi eutectic with and without silver as one alloy each)), and hold the board such that gravity will not pull the solder through the hole where it will pool in the plastic around the pin header - it will always do this to some extent, but you must ensure that that fugitive paste melts fully (so surface tension will pull it around the pin, rather than leaving a bunch of shorts).

Bismuth based solder is not recommended for this purpose unless you know the board is lead free. [the Solder guide (TLDR: forms ternary eutectic that melts below 100C).](Solder.md). This includes most HASL treated boards, which are leaded!

## Alignment
Unlike simple through-hole components, where you can insert the pins and then bend them over to hold them in place, pin header must be externally held in place. Sometimes you can do it with your hands, especially if the pin header piece is long (solder one end to hold it in place. Once the end is tacked in place. examine the board, and reflow the joint while pushing the header gently in the required direction if it's not straight, then do the rest. Wire solder is recommended, but paste can be used if and only if you are able to make sure it all melts, and doesn't trickle along the pin and pool in the plastic on the other side without the metal melting (and thus being sucked together by surface tension)

This method doesn't work if the header is very short (so you can't hold it without burning yourself) and doesn't work well if you have a row made up of several pieces (likely of different colors. There is a trick:

Order a few 2x40 pin SMT female pin header pices. insert the pin header into these, and put that into the board. It will hold the pins in perfect alignmment. If you have two rows, put a piece of male pin header betweeen the two in order to keep them the right distance apart, andhelp to ensure that they're really perfectly lined up.

On boards where there is to be an angled header between two straight ones (for example, my ATtiny 0/1/2-series breakout boards), there is a simple trick to holding those in place. Solder the pin headers ontothe edge first, then place the angled header in the holes, and tack it in place. then, if i's not straight, hold the board and put the female header onto the angled pins and use that to hold it while soldering it properly.

The same techniques work for female connectors, using male 2x40 SMT pin header. I can count on one hand the number of times I've actually soldered double row SMT pin header onto a board, but I use it all the time for alignment. The SMT version is preferred because the wider base makes it more stable, and can't get pushed up into the plastic body like normal female header can.

## Cutting
If there isn't going to be another piece of pin header right up against it, cutting male header is easy. You can usually just snap it, or use a pair of wire cutters on it.

If it's going to butt up against another piece of header (for example, if you want to make the power pins red and the ground pins green, like I do), you'll find that method to be insufficient - they won't fit, because the cutting process deforms the end, Or at least, not with most cutters. They make cutters based on a razor-like blade facing an anvil in a scissor type tool. Most reminscent of garden clippers, only much sharper. There is no consistent name for these, but they are made by many companies. Craftsman called them "Handi-Cut". These are a slightly frightening tool, as that blade is damned sharp (if it's not, replace the blade). Be careful when using it to keep it spelled "Handi Cut", not "Hand I Cut". These do a very good job of cutting header. The other option is to cut them the normal way, and then shave off the uneven end with a razor blade (I use a bare safety razor held between thumb and forefinger, holding the header against the worksurface and cutting down only to avoid injury.

Female pin header can never be cut gracefully. It is available in short lengths (in 6 colors too!). If you're deranged like I am, you might even choose to buy a selection of all the sizes the chinese sellers offer, in all colors. Then you'd have a few drawers full of them for when you whant them.

But yeah - if you want it to look decent, you need to get female headers of the length you need.

If you don't casre so much about appearances, though, you can take the 1x40 header, cut it in the middle of an unneeded pin, and then shave the end down to make it reasonably flat. It will still look like crap, but it works.

If you need to put two pieces of female header into adjacent holes, the edges will need to be shaved down a little with a razor - the length of the body of a female header is around `0.1 x number of pins + 0.02` - that is, it's about 10 mils too wide on each end to fit in adjacent holes on a PCB.

Some finer pitch pin header is NOT cuttable! You will likely need to try it in order to know if it is. The non-cuttable stuff will break wheere the pins are, instead of between them. Cut it longer and then pull out any unneded pins that survived that, and shave down the end with a razor - or just get some better header that can be cut.

## Angled, colored, female pin header
It's not available commercially - at least not at sane prices - but you can make it yourself. You get a piece of colored header of the desired length. use pliers to pull the pins out. Do the same to a piece of angled pin header, and then - you guessed it, push them intothe colored housing. They won't fit as securely, but that doesn't usually matter.

This is especially useful when using stackable pin header, as it is much cheaper per good pin to get 1x40 or 2x40 pins header, than to get smaller pieces. See [Stacking Pin Header][(StackingPinHeader.md)]
