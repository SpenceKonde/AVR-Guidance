Find and preparea location that you havr unrestricted write to and prepare a location there that you'll dump the files to. I have C:\data, and created c:\data\flashdumps to hold them.


Make sure you have enabled verbose upload. With the programmer connected to nothing, attmept to upload a skectch anyway. You will see juat vbefore the orange text starts in the console window, the command invocation that was used.


You get something like this:

```
C:\arduino-1.8.13\hardware\tools\avr/bin/avrdude -CC:\Users\Spence\Documents\Arduino\hardware\megaTinyCore\megaavr/avrdude.conf  -Ufuse2:w:0x01:m -Ufuse6:w:0x04:m -Ufuse8:w:0x00:m -Uflash:w:C:\Users\Spence\AppData\Local\Temp\arduino_build_474794/sketch_feb23a.ino.hex:i
```

It starts with two peices containging key pat to AVRdude and a good config file.
If you're on windows,you may have notices that not all of the slashes are in the same direction. Chabnge them all the backslashes,
These file locations will be vastly differrent for you.

```
C:\arduino-1.8.13\hardware\tools\avr\bin\avrdude -CC:\Users\Spence\Documents\Arduino\hardware\megaTinyCore\megaavr\avrdude.conf

```
You're not changing any fusses (I generated that with megatinyCore wwhich sets safe fuss on all uploads) so you don't need tthose directives. Abd your COM port is likely diferent - that should be pretty obvious.
```

C:\arduino-1.8.13\hardware\tools\avr\bin\avrdude -CC:\Users\Spence\Documents\Arduino\hardware\megaTinyCore\megaavr\avrdude.conf -v -patmega4809 -cjtag2updi -PCOM20 -b115200
```
Now we just point the flash directivbe at a location that is in our prepared folder and change the w to an r.
```
C:\arduino-1.8.13\hardware\tools\avr\bin\avrdude -CC:\Users\Spence\Documents\Arduino\hardware\megaTinyCore\megaavr\avrdude.conf -v -patmega4809 -cjtag2updi -PCOM19 -b115200 -Uflash:r:C:\data\flashdumps\testdump_2.hex:i
```
When I ran this on a AVR128DA64, it took me abou 20 seconds to dump the whole flash.

Similar process could be used to get the same SerialUPDI command by trying to upload to a DxCore or megaTinyCore part.

```
C:\Users\Spence\Documents\Arduino\hardware\megaTinyCore\megaavr\tools\python3\python3 -u C:\Users\Spence\Documents\Arduino\hardware\megaTinyCore\megaavr\tools\prog.py -t uart -u COM18 -b 230400 -d avr128da64 -fC:\data\flashdumps\test.hex -a read
```
The above command (dumping full flash of a 128k DA-series) did it in 8 seconds with default settings, or around 3s with aggressive settings.


Curiosity Nano, you figure would have to be fast, they manufacturer designed thee whole protocol, right? Nope  Speeds are - Serial UPDI > Optiboot > jtag2udpi > [NM]EDBG on every metric. :-P
