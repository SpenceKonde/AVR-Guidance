# Moving the USART pins
Sure, we all know you can swap the pins for the USARTs to an alternate set of pins using the PORTMUX.USARTROUTEx register - but maybe that isn't as flexible as you need (AVR DD-series is making the pin mapping more flexible judging by the product brief, though - it shows a ton of different pin mapping options for SPI0 and USART0 - but maybe this is only for the zeroth instance of a peripheral?). In any event, you can use the event system, in combination with the IRCOM mode (with pulse width decoding disabled) to move the RX pin around.

Similarly, the CCL mmodules can take as an input the TX signal from a USART (0, 1 or 2 only). In this way, you can (at least for the first three USARTs) move the pins around almost arbitrarily. 
