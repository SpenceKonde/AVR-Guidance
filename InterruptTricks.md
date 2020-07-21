# Interrupt tips and tricks
This is not yet organized - I'm just throwing things in this document when I discover them

## Interrupt Flags
The logic for when the proicessor calls an ISR is usually "If interrupt flag is set, while interrupt is enabled, and the global interrupt enable bit is set". Read the description of the interrupt flags associated with the interrupts you're writing carefully. On classic AVRs, these were almost always cleared by hardware. On the newer parts, Sometimes you have to explicitly clear them, or else the interrupt will continually retrigger, and in still other cases, they may be cleared by reading certain registers, that indicate you're doing something about the event that caused the interrupt.

## Empty interrupts
Often (eg, when waking up from sleep) an empty ISR is all you need. Use the EMPTY_INTERUPT(vectorname) macro instead of ISR(vectorname) with an empty code block. This saves a couple dozen bytes of flash and makes the interrupt finish doing nothing that much faster (an interrupt with an empty body will still get a "prologue" and "epilogue" generated, "clearing the decks" for, and then cleaning up after code that doesn't exist. 

