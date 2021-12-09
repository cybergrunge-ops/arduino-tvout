# Annotated TVOut library.

The goal of this repo is to provide a highly annotated version of the TVOut Library to allow beginners to understand how the library works, allowing easier experimentation, improvement and expansion of features. 

# Original Readme Text [avamander]

In order to use the library, place this library's folders (`TVout` and `TVoutfonts`) in your `libraries` folder. This is a library for generating composite video on an ATmega AVR microcontroller. The goal of this project is to create a simple interrupt driven library for generating composite video on a single AVR chip. Currently the output is NTSC or PAL at a resolution of 128x96 by default. The library currently works on ATmega168,328,1280,2560,644p,1284p,32U4,AT90USB1286 and more can be added by editing spec/hardware_setup.h. There are some timing issues with the m1284p, may be related to sanguino core.

## Connections

SYNC is on OCR1A and AUDIO is on OCR2A (except on the Arduino Leonardo, where AUDIO is on OCR0A). On NG, Decimila, UNO and Nano the sync is pin 9, video on 7 and audio on 11. On Mega2560	sync is pin 11, video is on A7(D29)	and audio is on pin 10.

# Microcontroller pins

MCU | SYNC | VIDEO | AUDIO | Arduino | SYNC | VIDEO | AUDIO
---|---|---|---|---|---|---|---
m168,m328 | B 1 | D 7 | B 3 | NG,Decimila,UNO | 9 | 7 | 11
m1280,m2560 | B 5 | A 7 | B 4 | Mega | 11 | A7(D29) | 10
m644,m1284p | D 5 | A 7 | D 7 | sanguino | 13 | A7(D24) | 8
m32u4 | B 5 | B 4 | B 7 | Leonardo | 9 | 8 | 11
AT90USB1286 | B 5 | F 7 | B 4 | -- | -- | -- | --

# Some notes on functioning from Myles

The "sync signal" pulls the video line to 0v, that is done by Timer1 compare pin (kind of like doing pwm). 
The "active video" signal portion sets the video line between 1v (white) and 0.3(v) black by toggling its state for each pixel.

Basically what happens:
Timer1 is setup to trigger an interrupt on each video line.
Timer1 sets its output compare io pin low during the sync pulse.
The timer1 compare match time is changed as needed to alternate between vertical and horizontal sync pulse lengths.

The interrupt does two things:
1. Call the hbi_hook, which is a function pointer for the user of the library to insert fast code
2. Call the line_handler, which is another function pointer that will run the code for each line of video output.

So you have 
blank_line: which is part of the "active" area of the display but with nothing on it, this is basically the only time user code gets to run.
active_line: which is the most complicated. 
    It has ensure cycle accurate timing (AVR instructions take between 1 and 3? cycles to execute, the interrupt has to wait for the instruction to finish) it does this with a call to wait_until which always returns at a set time into the video line.
    It then renders the line by calling the render_line function which is just a function pointer to one of the rendler_line6c 4c etc.
vsync_line: This manipulates the output compare time to the vsync pulse length, (it also generates sound but that can be ignored). At the end of the vsync lines it restores the output compare to the horizontal sync time.
Each of these handlers all count the lines and change the line_handler function pointer to the correct handler for that line number.

The render_line handlers all take the frame buffer and output it on the video pin one at a time. at 20mhz there is not enough time to create a more general output for x microseconds system so I created one for each one 6c,5c,4c are all essentially the same with different numbers of nop inserted to pad out the pixel display time (delay1, delay2, delay3, etc are just macros with some nop instructions). The render_line3c is destructive in that it does not have enough time to set the pin on the port so it sets the whole port.
 
One thing to note is that this basically all ignores interlacing and draws the same frame on odd and even frames.
