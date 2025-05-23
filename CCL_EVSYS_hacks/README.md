# CCL / Event System - hacks and tricks
This is a repository of clever tricks possible with CCL and/or the Event System - if you've done something cool with it... submit your demonstration code and explanation of what it does as a PR, and I'll gladly merge it. Looking for everything from basics to the bizarre. 


## Commentary "I wish I could..."
No system is perfect, even really awesome ones like this... Here are the things that I personally wish were possible
### EVSYS wishlist
* Register containing the (synchronized) values of the event channels. 1 byte, or 2 on 64-pin parts... and they already have synchronizers, so why not? 
* Did the event people forget about TWI? Considering what there ARE events for, it seems odd that there is no TWI involvement.
* Some event users require pulses at least as long as their (slower than CLK_PER) clock. Most pulse events are 1 CLK_PER. It'd be cool if their synchronizer latched so they didn't miss events in that case. This is particularly hard to generate with a software strobe! (I provide a workaround that does exactly that, a row of 20 st's, prefixed with logic to pick which of 5 points to jump into the stores at based on how long the user wanted the pulse)
### CCL wishlist (somewhat longer)
* 3 inputs is great. I never feel like I need a 4th input. Except when in2 is blocked up clocking the bloody thing. And then I feel handcuffed. 2 inputs is NOT enough, andthat's what's left with in2 clocking.
  * Any way to get a signal between the 32k internal and the system clock without using the third input! Even if I had to burn the CLKIN pin and set it to use an external clock!
* In SPI slave mode, I should be able 
* Register holding current output value of all LUTs subject to the usual sync delay. It's rather ridiculous that the only way we can find out what a CCL LUT is outputting is by directing it to a pin and reading the pin, just like with event system.
* The current sequential logic options aren't terribly compelling. They're not much more than you can do with a LUT.... Maybe a longer delay or shift register (bonus points if we can write it from software.  it)
* The PICs got 4-bit counters in their version of CCL?! When we gonna get them? (crowd booing at mention of PIC)
### One more thing
* Please give us some way to make the CCL's (or evsys, or both, whatever) output value control the DIRECTION of the pin, while leaving the value unchanged, so you could use this when you had to do open drain 
