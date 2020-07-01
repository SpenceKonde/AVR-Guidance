# The Magic of Lookup Tables

They may not be glamourous, and they may not be graceful, and they may not be the most flash-efficient way to address a challenge... but being both dead simple and highly performant, they can be a complete game changer when the situation calls for them. Since one is usually working with both input and output values that are highly constrained (for example, between 0 and 1023 for numbers from analogRead(), or 0 and 255 for numbers being output via analogWrite()), map() can also generate numbers outside of that range, requiring an additional check against minimum and maximum values. 

## map2lut tool
My map2lut tool provides a simple web interface for generating a lookup table from the values passed to map() - you just enter the values passed to map(), the minimum and maximum of the output, select the type, and click generate, and it will output valid C code that you can copy-and-paste into your sketch (or into a .h file that you include, if you prefer), as well as an example of accessing it. If you are using a processor that supports memory mapped flash access (megaAVR 0-series, like ATmega4809 (used in Uno WiFi Rev. 2 or Nano Every) or tinyAVR 0-series or 1-series, such as ATtiny3216), check the "memory mapped" box. 

## Case Study:
Read two analog values, scale them with map(), and output the resulting values to an MCP4912 10-bit DAC chip via SPI. Maximize the samples per second while using 16-ADC-clock sampling length, and 1MHz ADC clock. Using an ATtiny1614, ADC0 and ADC1 were configured in free-running mode, and SPI0 was configured for an 8MHz clock in master mode. Performance was initially profoundly disappointing - around a fifth of the theoretical speed! Two points of slowdown were identified during a study of performance, as shown in the table below. The simple nature of the code, obviously, contributed to the prominance of these performance impacts:

Samples/second | Description
--------|-------
~7900   | map() used to scale values, digitalWrite() used to manipulate SS pin
~9000   | map() used to scale values, direct port mamnipulation (fastDigitalWrite) used to manipulate SS pin
~23000  | lookup table used to scale values, digitalWrite() used to manipulate SS pin
~37000  | lookup table used to scale values, direct port mamnipulation (fastDigitalWrite) used to manipulate SS pin

In the final configuration, a 5:1 speedup was achieved by replacing digitalWrite() with direct port manipulation, and the map() call with an LUT; the latter accounted for 3-quarters of the improvement. This came at the cost of 2048 bytes of flash for each LUT - since the compiled size of the sketch was only 5.1K with the two LUTs, this was not a concern in this case (note that we did not have the option of backing off to an 814 to lower BOM costs - it lacks the second DAC which was crucial to both the high sample speed, and the elimination of channel switching error). 

This also underlines the miserable performance of digitalWrite() - this is due 
