# STK500 sync errors
These are a common - and unfortunately, very uninformative - error message to see when trying to upload. There is a lot of misinformation about these errors circulating. 

## Correct port?
Make sure you have the correct port selected
1. With the board (or serial adapter) not connected to USB, open tools -> port menu. Note what ports are listed. 
2. Connect the board (or serial adapter) via USB.
3. Open the tools -> port menu - you should see a new port. 
4. If you see a new port, that is the correct port to use. 
5. If you do not, it's a driver problem. Install the drivers for your board or serial adapter and try again.

## Correct board?
Verify that you have selected the correct board.
* Arduino Nano boards with two different bootloaders are circulating now. If using an Arduino Nano, make sure your official AVR board package is updated to where you see the Tools -> Processor menu with an option for 328p, and 328p (old bootloader). Try uploading with both of those settings
* "Arduino Due" is not short for Duemillanove. Support for the Arduino Due is in the official SAM boards package (it is not included with the IDE anymore)
* The Arduino Nano (sometimes called Arduino Nano 3.0) is not the same as the Arduino Nano Every (supported by official Arduino megaavr board package - or, for much better results, MegaCoreX). 
* The Arduino Nano Every is not the same as the Nano BLE or Nano IOT. These are also completely different boards, with different board manager packages for support.


## Anything connected to Reset, TX or RX pins?
(not applicable if using Arduino as ISP, see below section on that) Remove those connections - they can interfere with programming. Yes, this means that you may need to disconnect -> upload -> reconnect if your sketch uses the TX or RX pins, or you have anything more complex than a button connected to reset.


## It probably isn't drivers!
If you're at the point of getting that error, assuming you have the correct serial port selected, it is almost certainly NOT a driver problem. You can verify this with the Loopback Test:
1. Connect Reset and Ground on the board.
2. Connect TX and RX on the board.
3. Connect board via USB or whatever.
4. Open serial monitor, type something, and press enter. What you sent should be echoed back. 
If this passes, you *know* you have the right serial port selected, and you *know* the drivers are fine. 

If this fails, it could be drivers - but is more likely hardware problem!

## Is bootloader present and working?
Press reset. Onboard LED should blink (assuming the board has one). If it does not:
* if the board previously worked, it is probably damaged (either board, processor, or both - if the L LED is stuck on on a full-sized board, it is particularly likely that the board is damaged - replace). 
* If it hasn't been used, are you sure a bootloader is installed? Try bootloading it again with an ISP programmer. 

## When using Arduino as ISP

### You did remember to upload arduino as ISP to the Arduino you are using as programmer, right?
### Tools -> Board should be set to match the target, not the programmer
### You may need to disable autoreset (after uploading Arduino ISP sketch) 
Put a ~10uF capacitor between reset and ground. (any value from a couple uF up to basically any capacitance will work). Remove this before you try to upload a different sketch to the programmer.
### Leonardo/Micro may need a differnt tools -> programmer selection
### Naming is stupid
* The programmer sketch is called ArduinoISP
* The tools -> programmer entry is called Arduino As ISP
* The tools -> programmer entry called ArduinoISP is something completely different, don't use it.
### If you were able to upload ArduinoISP sketch, you know drivers, serial port, etc are correct
