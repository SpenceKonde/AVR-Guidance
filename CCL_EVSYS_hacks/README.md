# CCL / Event System - hacks and tricks
This is a repository of clever tricks possible with CCL and/or the Event System - if you've done something cool with it... submit your demonstration code and explanation of what it does as a PR, and I'll gladly merge it. Looking for everything from basics to the bizarre. 


## Commentary "I wish I could..."
No system is perfect, even really awesome ones like this... Here are the things that I personally wish were possible
### EVSYS wishlist
* Register containing the (synchronized) values of the event channels. 1 byte, or 2 on 64-pin parts... and they already have synchronizers, so 
* Did the event people forget about TWI? Considering what there ARE events for, it seems odd that there is no TWI involvement.
* Some event users require pulses at least as long as their (slower than CLK_PER) clock. Most pulse events are 1 CLK_PER. It'd be cool if their synchronizer latched so they didn't miss events in that case. This is particularly hard to generate with a software strobe! (I presume directing a pointer register to the software strobe address, and then stringing together as many ST intructions as needed would do it, but that gets inefficient real fast.
* Event users "SCK" and "XCK" for SPI and USART, working in EITHER MODE if it's master, that's what it will output as the clock, and if slave, that would be used instead of the XCK pin.
### CCL wishlist (somewhat longer)
* Feedback option for odd LUTs that gives them their own value, like even LUTs get.
* We have 3 bits for the clock source. and 3 of those 8 combinations aren't used. it's not like we don't have clocks that aren't an option... 
  * CLK_EXT
  * CLK_PLL
  * XOSC32k
* What will go into in the third synchronizer spot (it does nothing currently)?
* Instead of using LUT2 for the clock source, what about copying the same options as the other LUTs into that empty have of LUTnCTRL3? So we dont have to burn an input....
* Can we please get a way to use the output of the even LUT even when sequencer is being used? Why must it *replace* the even LUT output?
* CCL SPI inputs that work in master or slave mode - I want one that follows what we're outputting, and another that follows what we're taking in... Combined with the existing one for clock, there's inputs 0 through 2.
* Register holding current output value of all LUTs subject to the usual sync delay. It's rather ridiculous that the only way we can find out what a CCL LUT is outputting is by directing it to a pin and reading the pin, just like with event system.
* The current sequential logic options aren't terribly compelling - not enough to justify using to LUTs when many latch type operations can be performed by a single one. Surely we can come up with better? Maybe a delay with a bunch of stages, like a shift register - I have often wanted to be able to delay an output for longer than was otherwise practical.
### Could be in either place
* Open drain output, so that the event or CCL line would control direction instead of output value. 
