# Designing Libraries
Designing libraries presents a number of particular challenges. Of particular importance is careful design to ensure that it is easier to use than simple regiser manipulation. If your library doesn't make something easier than direct register manipulation, nobody will have reason to use it. At the same time, you must be very cognizent of ovehead. Overhead can usually be calculated with a formula something like this, (with different constants for RAM and Flash):
```text
Overhead = C + B (when only a single instance of peripheral being used or a C++ class based library for an external device)
Overhead = C + A*(instances of peripheral) = applicable to libraries like Logoc, Comparator, and such
Overhead = A + B + C + D... (for C library; eacj term representing one of the functions it provides.
```

If the library is for interfacing with external hardware, 

So you need to give most thought relating to overhead to the smallest flash parts that you want to be able to use your library, and to the parts with the most instances of the peripherals in question, and to the impact of moving from parts with 1 of a peripheral, to those with multiple. 

[!Library design, graphicall](tragicquadrant.png)


In the above diagram, for each specfic API function (for the Ardino API example), one could place a dot at a very specific point. likewise, for direct register writess, depeding on the peripheral, it may be straightforward to configure them by hand, or truly nightmarish. 

Any library, in order for users to adopt it, must be closer to the bottom of the chart or people will just use the maximally efficient, easier, register writes. A librar that fails this test should not be written nor used.

The challenge is how to stay in the lower left quadrant, withtout sacrificing flexibility - one could really extend this along the Z axis to cover functionality. Writing a simple, efficient library that uses a very powerful peripheral to do something very simple is far less of am accomplishment than one which is simple, efficient and exposes the true power of the peripheral or device in question/ 
