# Remarks and cautions relating to batteries from ebay, aliexpress, etc.

Rechargable LiPo battieries have a lot of things going for them. They're cheap, have a highly convenient output voltage, with output voltage essentially unchanged during discharge until the complete discharge, and are unriivaled for energy and power density by mass or by weight. They are the highest level of battery technology to see widespread use.

## With great power comes great responsibility
There are a few important issues to bear in mind with these batteries. The primary one being that their failure mode is particularly ungraceful... they tend to catch fire, and lithium metal fires are very difficult to extinguish (As an alkali metal, it reacts violently with water, releasing flamable hydrogen gas, and this reaction proceeds particularly readily if the metal is hot - such as, for example, when it is on fire. Do not attempt to use water to extinguish a burning lithium battery. This sort of catastrophic failure most often happens either in a highly degraded battery during charging, or when a battery is overcharged, or allowed to overheat. Occasionally poorly designed or manufactured batteries will suffer that sort of firey end even in the absence of abuse. Shorting batteries with no protection can overheat the battery through it's internal resistance and the waste heat of the chemical reaction.

Firey failures are most likely when the cell is fully charged.

## So how can I get some?

Well, you buy them. There are three general approaches here.

### Do: Buy quality cells from reputable distributors

18650s sort of leaked into common usage from industrial contexts, where users were sophisticated and knowledgable of how to assemble them into packs safely. This was done for laptop batteries for many years. Thus the initial supply chain was geared towards large MOQ's and batteries not ready for use as is. It was expected that buyers would be getting thousands, spotwelding them together into groups of a few to dozens, all connected to a battery management system. However, in the 2000's they began to make the rounds marketed to hobbyists and individuals (making huge flashlights was an early application, often using batteries with an integrated protection board), followed by the vape craze, which drove demand for the (slightly shorter) unprotected high current cells. It is now not as difficult to get legitimate batteries from real manufacturers.

### Do, but carefully: By quality cells from anonymous vendors
Since the supply chain for these batteries is not geared towards individuals buying just a few, you may be forced to.

### Don't: Buy too-good-to-be-true batteries from shady sellers on marketplace sites.
As these consumer uses proliferated, criminals saw an opportunity: Shrinkwrap could be printed cheaply and advertise a new brand and capacity, and discarded laptop batteries would provide the cells. Initially, plausible values were chosen, but that was soon abandoned, because an experienced buyer familiar with real cells, receiving batteries with an actual capacity of a couple hundred mAh (because old and degraded) labeled for 3000 mAh would quickly realize that they'd been had and complain - possibly before the crooks could withdraw their money. Someone who didn't know that a 5000 mAh 18650 was beyond current technology would be less likely recognize that the actual capacity wasn't even 500mAh, and it went down hill from there. Today, aliexpress and ebay are filled with 18650 batteries with fantastical capacities listed. 30,000 mAh?! 50,000?! The top of the line from the most advanced companies in the field in an 18650 battery is around 3500 mAh. Anyone claiming higher numbers is committing fraud, plain and simple, and they know it. Ultrafire, the most common fake battery brand, as far as I can tell, if it exists as an actual company at all, sells printed shrinkwraps, not batteries, and I'm pretty sure multiple companies are using the name now. These sellers are using the "nigerian prince" method: Their claimed specs are so crazy that anyone who might call them out would see the fraud immediately and not buy them in the first place - so the fraud is undetected for longer. Long enough for them to cash out their profits.

Usually sold for a dollar or less, these are not the wonder batteries claimed, but rather scrapped and discarded batteries. One of the big sources is laptop battery packs. You know that time when by the time you got rid of a laptop, the battery was the only part that worked? Neither do I. What about the last time the batteries held a charge long enough to carry between outlets? . By the time they end up shipped back to e-waste scrappers, those 18650 cells are lucky to store a few hundred mAh. Some of the fakes don't even HAVE 18650 cells in them, just steel cylinders in that shape with an itty bitty couple-hundred mAh battery stuffed inside and connected up.

### Don't: Buy anything that says "Ultrafire"
All of these are illegitimate, mismarked, and frequently non-functional, and potentially hazardous.

#### Safety implications
Bogus batteries present a hazard to more than just your wallet and schedule (when you have to place a second order after the first one turns out to be trash); it's a safety hazard as well (the Ultrafire branding is not inappropriate - though I'm not sure how much of a hazard these batteries are even able to pose anymore - many are are of such poor quality I doubt they could store the energy to burst into flame (Sorry, "Vent with flame" as they say in the battery business). The big concern you should have in mind (again, especially for 18650s) is that even if the sellers claim the batteries are protected, if the seller is willing to lie like that about their capacity, you can bet they'll say that "Oh yes these have overdischarge overcharge and short circuit protection, totally - and 50,000 mAh capacity!", regardless of whether or not it does. He may well not even know! Some do, some don't - it depends on whatever the batteries were originally in, before being scrapped and rewrapped, put a protection circuit in each cell, or had one protection board to handle all the cells.

## What do protection boards even look like? Are they easy to recognize?
Yes, actually  while you can't rule out their presence on cylindrical batteries without a dangerous white knuckle surgery, it's quite easy on flatpack batteries. The protection circuit normally consists of a TINY PCB, with 4 connection points (+ and + in and out per cell) containing two IC's. The first one is usually an SOT-23-6, and the second is either an SOT-23-6 for small, low current parts, and a small fine pitch MSOP variant where in the two center pins on each side of the chip (which has only 8 total) are soldered together. This is the actual switching element


### **Simple Battery chargers** are not protection boards
The typical simple battery charger will have one IC on the board and an assortment of passives.

They have 4 connection points: power out and power in, positive and negative.


### **Battery protect and charge boards**
These look just like common battery charging boards, except for on addition: An SOT-23-6 IC a and that wierd MSOP variant again. Yup - that part (likely a generic made by many companies) is ubiquitous.
