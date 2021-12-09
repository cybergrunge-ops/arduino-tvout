# (Egregiously) Annotated TVOut Library.

There is a very sore lack of well-documented and accessible libraries and projects for generating video signals using avr microcontrollers. The goal of this repo is to provide a highly annotated version of the TVOut Library to allow beginners to understand how the library works, allowing easier experimentation, improvement and expansion of features. It includes some 

# Table of contents

[CUE-f(E)AT-L: Cybergrunge-ops Ugly Explainer for (Egregiously) Annotated Tvout Library](https://github.com/cybergrunge-ops/arduino-tvout/blob/master/CUE-fEAT-L.md)

[Original Readme Text](#original-readme-text)

[Pins & Connections](#connections)

[Notes on functioning from Myles, 2021](#notes-on-functioning-from-myles-2021)

[Resources & Links](#resources-and-links)


# Original Readme Text

In order to use the library, place this library's folders (`TVout` and `TVoutfonts`) in your `libraries` folder. This is a library for generating composite video on an ATmega AVR microcontroller. The goal of this project is to create a simple interrupt driven library for generating composite video on a single AVR chip. Currently the output is NTSC or PAL at a resolution of 128x96 by default. The library currently works on ATmega168,328,1280,2560,644p,1284p,32U4,AT90USB1286 and more can be added by editing spec/hardware_setup.h. There are some timing issues with the m1284p, may be related to sanguino core.

# Connections

SYNC is on OCR1A and AUDIO is on OCR2A (except on the Arduino Leonardo, where AUDIO is on OCR0A). On NG, Decimila, UNO and Nano the sync is pin 9, video on 7 and audio on 11. On Mega2560	sync is pin 11, video is on A7(D29)	and audio is on pin 10.

MCU | SYNC | VIDEO | AUDIO | Arduino | SYNC | VIDEO | AUDIO
---|---|---|---|---|---|---|---
m168,m328 | B 1 | D 7 | B 3 | NG,Decimila,UNO | 9 | 7 | 11
m1280,m2560 | B 5 | A 7 | B 4 | Mega | 11 | A7(D29) | 10
m644,m1284p | D 5 | A 7 | D 7 | sanguino | 13 | A7(D24) | 8
m32u4 | B 5 | B 4 | B 7 | Leonardo | 9 | 8 | 11
AT90USB1286 | B 5 | F 7 | B 4 | -- | -- | -- | --
 

# Notes on functioning from Myles, 2021

NOTE: This library basically ignores interlacing and draws the same frame on odd and even frames.

The "sync signal" pulls the video line to 0v, that is done by Timer1 compare pin (kind of like doing pwm). 
The "active video" signal portion sets the video line between 1v (white) and 0.3(v) black by toggling its state for each pixel.

Basically what happens:
Timer1 is setup to trigger an interrupt on each video line.
Timer1 sets its output compare io pin low during the sync pulse.
The timer1 compare match time is changed as needed to alternate between vertical and horizontal sync pulse lengths.

The interrupt does two things:
1. Call the `hbi_hook`, which is a function pointer for the user of the library to insert fast code
2. Call the `line_handler`, which is another function pointer that will run the code for each line of video output.

So you have 
`blank_line`: which is part of the "active" area of the display but with nothing on it, this is basically the only time user code gets to run.

`active_line`: which is the most complicated. It has to ensure cycle accurate timing (AVR instructions take between 1 and 3? cycles to execute, the interrupt has to wait for the instruction to finish) it does this with a call to `wait_until` which always returns at a set time into the video line. It then renders the line by calling the `render_line` function which is just a function pointer to one of the `render_line6c`, `4c`, etc.

`vsync_line`: This manipulates the output compare time to the vsync pulse length, (also generates sound). At the end of the vsync lines it restores the output compare to the horizontal sync time.

Each of these handlers all count the lines and change the line_handler function pointer to the correct handler for that line number.

The `render_line` handlers all take the frame buffer and output it on the video pin one at a time. 

At 20mhz there is not enough time to create a more general output for x microseconds system so I created one for each one `6c`,`5c`,`4c` are all essentially the same with different numbers of `nop` inserted to pad out the pixel display time (`delay1`, `delay2`, `delay3`, etc are just macros with some `nop` instructions). The `render_line3c` is destructive in that it does not have enough time to set the pin on the port so it sets the whole port.

# Resources and links

## Resources about ASM 

TVout uses assembly language to directly manipulate the ports for sync pin and video pin. For beginners (like me) this assembly language is intimidating when you first look at it without knowing anything about asm. In the interest of making things easier for others, i have been doing a ton of research into assembly language, i will link here some resources that helped me a lot:

https://ucexperiment.wordpress.com/2016/03/04/arduino-inline-assembly-tutorial-1/
this is a great intro to asm that is specific to arduino. it is written pretty much in plain english that is easy to understand.

https://gcc.gnu.org/onlinedocs/gcc-11.2.0/gcc/Using-Assembly-Language-with-C.html#Using-Assembly-Language-with-C
This is the official GCC guide on asm in C. GCC is the compiler used by arduino. This stuff is a bit more obtuse to read but since it is straight from the GNU's mouth.... it is crucial to read.

viznut.fi/texts-en/machine_code.pdf
Viznut wrote a great intro to asm here

https://www.nongnu.org/avr-libc/user-manual/inline_asm.html
https://www.nongnu.org/avr-libc/user-manual/assembler.html
AVR-LibC specific manual on asm.

## Resources about timers, interupts and registers

TVOut uses direct manipulation of the timers and ports, and understanding how it works requires one understand how microcontrollers' Timers and Counters work.
The biggest barrier for me in getting to understand timers, registers and counters was the fact that they involve all kinds of isoteric naming conventions, so you have to memorize what various things mean, for instance such obscure and occult names like OCRA1, TCCR1A, COM1A0, CS10, WGM12, ICR1, TOIE1, and the classic TIMSK1, among others. These names are all very funny to try to pronounce out loud, and aside from learning what they do, much Elocutionary Entertainment can be had. however, to get to the point, below i have listed some resources on how timers and all these other things work.

https://www.gammon.com.au/timers
https://gammon.com.au/interrupts
Gammon.com.au is an invaluable resource, and Nick explains things very well, as well as providing some basic example programs for use with arduino!

