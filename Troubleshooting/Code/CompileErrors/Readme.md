# Troubleshooting Your Sketch
This document describes some techniques that can be helpful for finding and/or understanding bugs that cause compile errors that don't make sense, at least initially. In some instances, some thoughts on how one might go about fixing them. This section is for *compile* errors, where even "Verify" produces an error. If that works, you want the (TODO: Other section).

# So the sketch won't compile?
Be sure to expand the console and copy+paste ALL the errors into a text file - often the thing that *actually* generated the error was listed further up, and caused a cascade of errors, ending in one far removed from whatever caused it. Also, look for any Warnings - these are things that aren't errors, but may hint at where a bug is coming from (or may be less of a concern). Make sure you enabled warnings in File -> Preferences. 

Verbose compile output is often not very useful if it's just one sketch that's broken - it outputs a lot of text, but it's often not useful - and it can bury useful warnings under a wall of text. I suggest keeping it off when the problem is just with your sketch. If Bare Minimum doesn't compile - either for one board, or in general, there is a larger issue, and often verbose output will contain some additional information. At worst, that's the information you'll be asked for if you post on forums asking for help. 

This guide does not cover basic syntax errors, though it may mention them in passing, particularly ones that are associated with problems that are often the subject of posts on the Arduino forums. I try to cover interesting and/or Arduino specific issues (as well as giving obligatory mentions to a few VERY common ones) here.

## Does Bare Minimum compile? 
Does the Bare Minimum example compile for the part you're using? 

If not, does it compile for the Arduino Uno? If it compiles for the Arduino Uno, but not the part you want, there is a problem with the board package or "core" or it is installed incorrectly. not your sketch.

If it is broken for the Arduino Uno too, your IDE is broken. See (TO DO: Nothing compiles)

If Bare Minimum does compile for the part you want, but your code doesn't - but you just can't figure out why - this may be useful. 

## This is a work in progress and far from complete!

## Specific errors and the more interesting things they can be caused by
This is the meat of this guide, or it will be: Specific errors, and suggestions on what to do when it's not something simple. 

### 'variable' was not declared in this scope 
**Most often, this is simply an typo in the name of a variable, or you forgot the rules of variable scope in C (google it or go to your favorite general C/C++ reference website for a refresher).** 

When this is coming from a library (typically, one that you know works for other hardware) or in low-level code (which you may not have written or understand - a common situation in Arduino), that "variable" may actually a hardware register that simply doesn't exist on the part you're compiling for - either it doesn't have that register (or the capabilities associated with it) or it's named something different. If the name has all CAPITAL letters and/or numbers, possibly with underscores or a dot, and is abbreviated aggressively, that's probably a register name. See (MissingRegister.md)

### Multiple definition of ...
This is usually a simple one: you're attempting to define the same variable or function twice. The compiler will tell you where the two declarations are. 

If it's in a file called `iosomething.h`, you've picked a register name. Name it something different. Also, don't use `PC` or `SP` - those are the Program Counter and Stack Pointer, respectively, which should not be modified directly. Variables in general should not be named something that could be a register name.

