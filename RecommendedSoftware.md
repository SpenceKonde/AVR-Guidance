# Recommended Tools (software)
In addition to the Arduino IDE (or your choice of other IDEs - comparison of IDE options is beyond the scope of this guide), there are a few other tools that can be of great use when developing for embedded systems. 

## A good text editor
A good text editor that provides syntax highlighting for .c, .cpp, and .h files is invaluble. While you have the IDE to provide this functionality for the code you are writing, that is far from the only source code you will be looking at. For AVRs, anyone writing anything that goes beyond the simple Arduino API will end up looking at the "io" header files, at minimum, and likely many other files. There are a great many options here - it's the sort of "holy war" type choice where, for many programers, the only right answer is the one they agree with. Personally, I like Sublime Text. For Windows users, Notepad++ is another good option. I won't take sides here - the important thing is to find one that works well and that you're happy with. 

## A good serial console
The Arduino Serial Monitor is NOT a good serial console. I recommend hTerm (which is written by an embedded developer) - it has more features than a swiss army knife, and does all of them very well. Some particularly useful features include
* Configure what, if anything, to send when you press enter (CR, LF, CR+LF, or nothing).
* Manual control over DTR and RTS pin (when "autoreset" circuit is present, DTR (or sometimes, RTS) will reset the board. You want to control this manually, instead of it being done automatically when you open the connection)
* Show status of CTS, DSR, DCD and RI pins (This allows you to see the state of up to four pins on your screen, instead of having to wire up LEDs and look away at them during testing.
* Display - and enter - characters as ASCII (text), hex, decimal, or binary, ideally showing more than one representation of the data at once. Protocols that send raw binary data over serial are very common in embedded systems, but few serial consoles make it easy to view this, and even fewer show more than one representation side by side. 

## Python
A great scripting language. Why do you need this, when you're writing embedded C? The intelhex and pyserial modules, combined with the ability to call other applications allows you to easily merge hex files, compile sketches, and so-on is incredibly useful. Better still, it is very easy to pick up and do stuff with without having to actually *learn* python. Some things I have done with it:
* Automatically generate configuration files for large numbers of parts
* Merge, edit, and compare hex files easily
* Automate uploading of hex files, including after making modifications (for example, to include a serial number)
* Compare io header files for different parts, and generate listing of which features are supported by which parts
* Automatically compile and upload a sketch with various settings (via Arduino CLI)
* Process log files
In one case (sadly, the code was lost to a hard drive failure - always check your code into github or otherwise back it up), I automatically compiled and uploaded a "millis/micros test" with each available timer selected as the source for millis/micros timekeeping, recorded its output, and processed that output to record whether any problems were found, and generate a .csv file for further investigation in Excel. Tnis was used to detect and fix numerous problems with the millis/micros timekeeping on megaTinyCore 
