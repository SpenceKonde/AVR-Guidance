# Prescaling clocks with the CCL
A CCL LUT can be put into a configuration where it will oscillate as rapidly as it's transistors can switch (this is extremely quickly); on it's own, this is not of much use. However, it has two "clocked" options, a synchronizer and a filter, which respectively introduce a delay of 2 or 4 clock cycles per transition. Combined, an oscillating configuration with one of those enabled will prescale the clock source being used. Why would one want that? The first use that comes to mind is generating a prescaler for a type B timer that differs from that used by the type A timer(s), or which is not an option on those timers at all. 

Not only that, if using a clock source other than the internal oscillator and a device that supports using that as the clock source, you can generate frequencies unrelated to the system clock... and use them to clock a type B timer (though since type B timers themselves are synchronous, you must be mindful of the nyquest criteria (prescale appropriately), and each tick would be a multiple of system clock cycles, but the average would be this exotic frequency. Considering that the tinyAVR 2-series internal oscillator looks likely to match the incredible compliance of the 0/1-series one (typically from 5/8ths of the nominal clock speed to 1-5/8ths of it) looks set to be a particulary flexible chip in this regard. 

These demonstrations use the Logic library, for readability and clarity. Event configuration shown only for non tinyAVR 0/1-series for brevity. adjust as appropriate.

## Simple /4 or /8
Using the EVEN logic block is easier, since feedback works.

```
  Logic0.enable = true;               // Enable logic block 0
  Logic0.input0 = in::feedback;       // Use event channel a
  Logic0.input1 = in::masked;         // masked
  Logic0.input2 = in::masked;         // masked
  Logic0.output = out::enable;        // Enable logic block 0 output pin PA3 (Dx) or PA4 (tiny)
  Logic0.filter = filter::sync;       // sync = /4, filter = /8
  Logic0.truth = 0x01;                // Set truth table - HIGH if input LOW, (and the other two are masked off)
  // Initialize logic block 0
  Logic0.init();
  Logic::start();
  
```

Using any logic block, and an event channel
```
  Logic1.enable = true;               // Enable logic block 0
  EVSYS.CHANNEL0 = EVSYS_CHANNEL0_CCL_LUT1_gc;
  EVSYS.USERCCLLUT1A = EVSYS_USER_CHANNEL0_gc;
  Logic1.input0 = in::event_a;        // Use event channel a instead of feedback
  Logic1.input1 = in::masked;         // masked
  Logic1.input2 = in::masked;         // masked
  Logic1.output = out::enable;        // Enable logic block 1 output pin
  Logic1.filter = filter::sync;       // sync = /4, filter = /8
  Logic1.truth = 0x01;                // Set truth table - HIGH if input LOW, (and the other two are masked off)
  // Initialize logic block 0
  Logic1.init();
  Logic::start();
  
```

## LUT pair for /12 or /16
Two filters give /16, sync and filter give /12. 
Can extend further, using linked LUTs for input; prescaling is added.

```
  Logic0.enable = true;               // Enable logic block 0
  Logic0.input0 = in::link;           // Use output of block 1
  Logic0.input1 = in::masked;         // masked
  Logic0.input2 = in::masked;         // masked
  Logic0.output = out::enable;        // Enable logic block 0 output pin or PA4 (ATtiny))
  Logic0.filter = filter::sync;       // No output filter enabled
  Logic0.truth = 0x01;                // Invert output of linked LUT
  // Initialize logic block 0
  Logic0.init();

  Logic1.enable = true;               // Enable logic block 1
  Logic1.input0 = in::feedback;       // use output of even-numbered block, ie, block 0
                                      // On 0/1-series tiny, link also works.
  Logic1.input1 = in::masked;         // masked
  Logic1.input2 = in::masked;         // masked
  Logic1.output = out::enable;        // enable logic block 1 output pin
  Logic1.filter = filter::filter;     // No output filter enabled
  Logic1.truth = 0x02;                // Set truth table - just copy (and delay) what first one was outputting
  Logic1.init();
  Logic::start();
  ```
  
  ## LUT pair for /32 or /64
  Here, we have both LUTs configured as if they were prescaling alone - but use the linked LUT as the clock source. Prescale factors now multiply, as above, event channel needed for the odd LUT. 
  
  ```
  Logic0.enable = true;               // Enable logic block 0
  Logic0.input0 = in::feedback;       // Use event channel a
  Logic0.input1 = in::masked;         // masked
  Logic0.input2 = in::link;           // used for clock
  Logic0.output = out::enable;        // Enable logic block 0 output pin PA3 (Dx) or PA4 (tiny)
  Logic0.filter = filter::filter;     // sync = /4, filter = /8
  Logic0.clocksource = clocksource::in2; // use input 2 as the clock
  Logic0.truth = 0x01;                // Set truth table - HIGH if input LOW, (and the other two are masked off)
  // Initialize logic block 0
  Logic0.init();
  
  Logic1.enable = true;               // Enable logic block 0
  EVSYS.CHANNEL0 = EVSYS_CHANNEL0_CCL_LUT1_gc;
  EVSYS.USERCCLLUT1A = EVSYS_USER_CHANNEL0_gc;
  Logic1.input0 = in::event_a;        // Use event channel a instead of feedback
  Logic1.input1 = in::masked;         // masked
  Logic1.input2 = in::masked;         // masked
  Logic1.output = out::enable;        // Enable logic block 1 output pin
  Logic1.filter = filter::filter;     // sync = /4, filter = /8
  Logic1.truth = 0x01;                // Set truth table - HIGH if input LOW, (and the other two are masked off)
  // Initialize logic block 0
  Logic1.init();
  Logic::start();
```

## Extending further
There is an incredible variety possible here... Consider something like this:



  ```
  // LUT2 and 3 doing /3
  
  Logic2.enable = true;               // Enable logic block 2
  Logic2.input0 = in::link;           // LUT3 output
  Logic2.input1 = in::masked;         // masked
  Logic2.input2 = in::masked;         // masked
  Logic2.output = out::enable;        // debug?
  Logic2.filter = filter::sync;       // sync = /4, filter = /8
  Logic2.truth = 0x01;                // Set truth table - HIGH if input LOW, (and the other two are masked off)
  // Initialize logic block 0
  Logic2.init();
  
  Logic3.enable = true;               // Enable logic block 3
  Logic3.input0 = in::feedback;       // Use LUT2's output
  Logic3.input1 = in::masked;         // masked
  Logic3.input2 = in::masked;         // masked
  Logic3.output = out::enable;        // debug?
  Logic3.filter = filter::filter;     // sync = /4, filter = /8
  Logic3.truth = 0x02;                // Set truth table - HIGH if input LOW, (and the other two are masked off)
  // Initialize logic block 0
  Logic3.init();

  Logic1.enable = true;               // Enable logic block 1
  Logic1.input0 = in::feedback;       // use output of even-numbered block, ie, block 0
  Logic1.input1 = in::masked;         // masked
  Logic1.input2 = in::link;           // link for clock - that's LUT2's /12-prescaled one
  Logic1.clocksource = clocksource::in2; // use input 2 as the clock
  Logic1.filter = filter::filter;     // No output filter enabled
  Logic1.truth = 0x02;                // Set truth table - just copy (and delay) what first one was outputting
  Logic1.init();
  
  Logic0.enable = true;               // Enable logic block 0
  Logic0.input0 = in::link;           // Use output of block 1
  Logic0.input1 = in::masked;         // masked
  Logic0.input2 = in::??????          // Could use event to get the /12 clock - thus making this a /144 prescale
                                      // OR mask it off, and just add 4 clocks giving a /100 prescaler! 
  Logic0.clocksource = clocksource::  // either unneeded, or in2... 
  Logic0.filter = filter::sync;       // No output filter enabled
  Logic0.truth = 0x01;                // Invert output of linked LUT
  
  Logic0.init();
  Logic::start();
  
```

Yeah, it can get damned weird... **and the Dx-series parts have 6 LUTs** 
