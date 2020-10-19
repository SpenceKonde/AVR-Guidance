# Surprising compile errors
It's always good when you get a compile error when there's a problem with your code - not as good as if everything works, but preferable to code that compiles, but does something different than what you expect. Many compile errors are clear and helpful, pointing you right to where the problem is, often with a description that matches what's wrong. Others are less illuminating. 

## General Notes
* If it doesn't make sense, look at the previous couple of lines - the compiler flags the point where it first realized that there was a problem... 
* Missing semicolons and mismatched paren/braces/brackets often make very strange errors.

## Specific errors
These are particularly obtuse errors I've seen personally

### error: expression cannot be used as a function

```
C:\Users\Spence\Documents\Arduino\sketches\ClockFromNano\ClockFromNano.ino: In function 'void setup()':
ClockFromNano:4:3: error: expression cannot be used as a function
   DDRB=0x26; //OC1B pin and LED set output.
   ^
```
Huh? I'm just trying to assign a value to a register! Actual problem? A missing semicolon at end of previous line - no clue how to compiler got that error message out of it...

