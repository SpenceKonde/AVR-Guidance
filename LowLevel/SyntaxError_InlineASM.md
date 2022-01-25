# The error says there's a problem, but it points me to a temp file that gets instantly deleted

Yes, yes it does, and it's quite obnoxious that it leaves you with no debugging info isn't it, not even a clue as to what part of the file, (at least you know what file it came from!)
And that's actually very important to getting some information. With verbose compiler output enabled, attempt to compile and find the avr-gcc or avr-g++ line that is compiling the file

```
"C:\\arduino-1.8.13\\hardware\\tools\\avr/bin/avr-g++" -c -g -Os -Wall -std=c++17 -fpermissive -Wno-sized-deallocation 
-fno-exceptions -ffunction-sections -fdata-sections -fno-threadsafe-statics -Wno-error=narrowing -MMD -flto -mrelax 
-mmcu=attiny84 -DF_CPU=16000000L -DCLOCK_SOURCE=0x1 -DARDUINO=10813 -DARDUINO=10813 -DARDUINO_AVR_ATTINYX4 -DARDUINO_ARCH_AVR 
"-DATTINYCORE=\"2.0.0-dev\"" -DATTINYCORE_MAJOR=2UL -DATTINYCORE_MINOR=0UL -DATTINYCORE_PATCH=0UL -DATTINYCORE_RELEASED=0 
"-IC:\\Users\\Spence\\Documents\\Arduino\\hardware\\ATTinyCore\\avr\\cores\\tiny" 
"-IC:\\Users\\Spence\\Documents\\Arduino\\hardware\\ATTinyCore\\avr\\variants\\tinyx4_cw" 
"C:\\Users\\Spence\\Documents\\Arduino\\hardware\\ATTinyCore\\avr\\cores\\tiny\\TinySoftwareSerial.cpp" 
-o "C:\\Users\\Spence\\AppData\\Local\\Temp\\arduino_build_421258\\core\\TinySoftwareSerial.cpp.o"
```
In reality that whole mess is all on one line, but code blocks on github markdown documents don't autowrap, and it was totally unreadable like that

If you're on Windows, you need to find-replace the `\\` and `/` with `\` otherwise the paths will choke it. 

Three changes need to be made after tbat:
1. Change the `-c` to a `-S` to tell it to output generated assembly. 
2. Change the final parameter, the path after the -o, to a file located somewhere convenient, preferably with an extension that will open in your text editor. 
  That command will run, and it will produce output - but it's total gibberish. Where's my assembly? 
3. Remove the -flto - the link time optimization causes it to use a different format for the otutput, and a completely uninteligible one. 

You get something that looks like this (except that it's all on one line)
```
"C:\arduino-1.8.13\hardware\tools\avr\bin\avr-g++" -S -g -Os -Wall -std=c++17 -fpermissive -Wno-sized-deallocation 
-fno-exceptions -ffunction-sections -fdata-sections -fno-threadsafe-statics -Wno-error=narrowing -MMD -mrelax 
-mmcu=attiny84 -DF_CPU=16000000UL -DCLOCK_SOURCE=1 -DARDUINO=10813 -DARDUINO=10813 -DARDUINO_AVR_ATTINYX4 
-DARDUINO_ARCH_AVR "-DATTINYCORE=\"2.0.0-dev\"" -DATTINYCORE_MAJOR=2UL -DATTINYCORE_MINOR=0UL -DATTINYCORE_PATCH=0UL 
-DATTINYCORE_RELEASED=0 "-IC:\Users\Spence\Documents\Arduino\hardware\ATTinyCore\avr\cores\tiny" 
"-IC:\Users\Spence\Documents\Arduino\hardware\ATTinyCore\avr\variants\tinyx4_cw" 
"C:\Users\Spence\Documents\Arduino\hardware\ATTinyCore\avr\cores\tiny\TinySoftwareSerial.cpp" -o "C:\data\asmbug.S"
```
Of course, since you didn't use link time optimization, the line numbers are all different now.... and it's about as ugly as it gets, but at least now you can look at what the compiler is looking at. 

In the case of problem from this example, I was getting a "garbage at end of line" error from line 165 of the temp file. The error was found on line 187 of the asmbug.S file:

```
	com r22
	sec
	_txstart:brcc _txpart2
	out 27, r25
	rjmp .+4_txpart2:out 27, r24
	nop
	_txdelay:rcall uartDelay
	rcall uartDelay
	rcall uartDelay
	rcall uartDelay
	lsr r22
	dec r19
	brne _txstart
  ```
  The third line there consists of both the third and fourth line, as I'd forgotten the `\n\t` on one line. 
