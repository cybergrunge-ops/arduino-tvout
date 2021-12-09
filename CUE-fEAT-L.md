
# Cybergrunge-ops' Ugly Explainer for EgregiouslyAnnotatedTvout-Library


## Intro

I am Ellie Voyyd she/her, founder of [cybergrunge.net](https://cybergrunge.net). I am an Outsider Engineer, I am self-taught due to disability interfering with my ability to work and participate in school. So, this all might seem weird to "traditional" engineers and programmers. I also am somewhat insane and diagnosed with major depression, schizophrenia and several other things. Anyway, moving on..

# What Is TVout?

TVout was made by Myles Metzler a decade or so ago. It uses various [8-bit](https://en.wikipedia.org/wiki/8-bit_computing) AVR [microcontrollers](https://en.wikipedia.org/wiki/Microcontroller) to output monochrome [NTSC](https://en.wikipedia.org/wiki/NTSC) and [PAL](https://en.wikipedia.org/wiki/PAL) signals, and allows some very simple functionality like sound generation, drawing simple shapes and lines, bitmaps and so on. The library uses a few conventions and methods that I am not very familiar with, like [inline assembly](https://ucexperiment.wordpress.com/2016/03/04/arduino-inline-assembly-tutorial-1/), [bitwise operations](https://youtu.be/YJgnYQEhYBw) and [pointers](https://www.gammon.com.au/callbacks). These make it a bit less [accessible](https://en.wikipedia.org/wiki/Accessibility) to newbies, but hopefully i can explain in the commentary what is going on, step by step.

# Beginning Our Analysis 

In TVout, we have four folders: `examples`, `pollserial`, `spec`, and `TVoutfonts`. We have nine files in this root directory. `keywords.txt`, `library.properties`, `version history.txt` (these can be ignored right now). Then we have:`TVout.h`, `TVout.cpp`, `video_gen.h`, `video_gen.cpp`, and `TVoutPrint.cpp`. These are the ones that we will describe first because they do most of the work.

For those unfamiliar with C and C++, note that that there is the code files, which are `.cpp`, and there are header files, which are `.h`. Header files are used "to define the physical layout of program data, pieces of procedural code and/or declarations". The use of headers and modularity are little fancy things that programmers do to make their programs and libraries easier to code for. Keep in mind that everything done in here could be done in a bare `.c` file, but things are separated out to organize things. For my level of understanding, this just adds unnecessary complexity and more vectors for confusion... but whatever.

## TVout.h

Lets first look at `TVout.h` and `TVout.cpp`, since thats what the library is named. These don't generate any video directly, they are just the basis for the library. You will know from the examples that you include the library by typing `#include <TVout.h>`, you then define TV by going `TVout TV;`, to access its functions such as `TV.begin();`, `TV.set_pixel();`.

In the header we have includes for general avr stuff, and then some includes for the library itself. After these, we have some defines which is basically just a fancy way of renaming or assigning nicknames to different values and functions. After this, we have the class being constructed called `TVout`. This is very sensible, and so far everything is very nice and makes a lot of sense. Inside TVout, we have `uint * screen`. Why is there an asterisk? Well, apparently this is used in C and C++ for something called a callback, or a pointer. This is confusing and stupid and i literally could not care less what a callback is at this point.

Next we have a bunch of methods that can be used by TVout, most of which are self-explanatory: `set_pixel()`,  `get_pixel()`,  `fill()`, `draw_line()`,  `shift()`,  `draw_row()`, `draw_column()`,  `get_pixel()`,  `draw_rect()`, `draw_circle()`,  `bitmap()`,  `print()`, `select_font()`, `tone()`,  `noTone()`,  `print_char()`, `set_cursor()`,  `write()`.

We had some methods that were a bit more obscure but still pretty self-explanatory. `set_vbi_hook()` is used to insert any function you want inside the vertical blanking interval. Likewise, `set_hbi_hook()` lets you insert any function you want to occur inside every horizontal blanking interval. Miles notes to us that you have to make sure that the function is not too long and clunky, otherwise it will mess up the timing of the video output. After all this we have some private methods. What is a private method? who knows it probly doesnt matter.

Well, there we go! We now understand TVout completely and there is nothing left to do! :D

## TVout.cpp

Ah, well i guess i was wrong. There is more that needs to be understood. As i said, there are five significant files in the root directory of TVout, we have only looked at one of them. Since we looked at `TVout.h`, lets look at its accompanying `.cpp` file. Upon opening it in text editor we see some comments, license, include TVout.h, bla bla, `TVout::begin(uint8_t mode)` to call with default resolution, blabla. We can here see the actual C code for all of the methods listed above. That is all very nice, but we want to know how the video is being actually generated. So, there is nothing useful here for us, we will have to look deeper into the library.

## video_gen.h, video_gen.cpp

Let us return to `TVout.h` for a moment and notice that there was included a file called `video_gen.h`. Since we want to know how TVout generates video, this file seems like it might be relevant :) Let us open it and see what is in it, since we know that `.h` files are headers which give a nice overview of what can be done. Let us also open `video_gen.cpp` so that if we wonder what is going on, we can see the actual code.

As we enter this new territory of `video_gen.h`, we are greeted by the familiar license and definition, followed by something called a `typedef struct`. This is a very arcane sounding thing, and inside of it there is some stuff that looks pretty normal. Everything else seems pretty normal, so maybe this header is not very useful for understanding what is going on, lets make a plan to visit the domain of the `.cpp` file. On our way out, we do notice a `foreboding` reference to some things which are named `render_line6c()`, `render_line5c(),`, `render_line4c(),` and `render_line3c(),`. Seeing these strange references fills us with a very vague sense of, not dread per se, but unease... Why does the numbering start at six? why does it only count down to 3? In total there are four of these `render_line` functions. ... Four... Three... Six..... What could this mean? As we open the doors of `video_gen.cpp` we see the license. We then see some includes. Refered are some documents in the folder `spec`. Our prior sense of unease is piqued by the presence of some inactive, commented defines for REMOVE6C, 5C, 4C, 3C. What is 6c, 5c, 4c, and 3c? Why are they being removed (or, in this case.... NOT removed)? We could just ignore them, but something gives us a feeling that they are important.

Looking ahead, we see some pointers like `vbi_hook` and `render_line`. We notice that these are new and different, and that there is some strange use of parenthesis and a strange ampersand in such lines as `void (*hbi_hook)() = &empty;`. I think this has to do with pointers. Asterisks and ampersands in front of names have something to do with pointers. i don't know anything about pointers and have no interest in learning, so lets move on.

We now come to `render_setup()`, a friendly seeming function. Lets analyze what it does.

`render_setup()` requires a mode, an hres and vres, and a weird thing called `*scrnptr`, which seems to be another `pointer thingy` or `whatever`. We see a nice if/else for PAL and NTSC mode. We have a breifly commentated initialization of an `rmethod`, immediately followed by a switch/case which is neatly tied up. This seems to just be selecting different modes of display, and for most purposes, we will just be using a default mode of NTSC or PAL (NTSC in my instance), so it doesnt seem very important. However, we do note here some references to those mysterious `render_linec` functions... Even more ominous, they are here presented with ampersands in front of them, which seems to indicate some kind of pointer...  Let us move on, after taking a moment to pray that we will not have to learn what pointers are...

Next we see this:

```	
  DDR_VID |= _BV(VID_PIN);
	DDR_SYNC |= _BV(SYNC_PIN);
	PORT_VID &= ~_BV(VID_PIN);
	PORT_SYNC |= _BV(SYNC_PIN);
	DDR_SND |= _BV(SND_PIN);	// for tone generation.
```
this is pretty normal. moving on.

Now we have a simple two-liner to set up inverted fast pwm mode on the timer.
```
	TCCR1A = _BV(COM1A1) | _BV(COM1A0) | _BV(WGM11);
	TCCR1B = _BV(WGM13) | _BV(WGM12) | _BV(CS10);
```
this is normal. after this we have an if/else to `start_render` in NTSC or PAL. We should note the references here to `ICR1` and `OCR1A`, which are Registers, that have to do with outputs. after this stuff, we close with some `normal stuff`, involving more pointers we will ignore. lets note that `TIMSK1 = _BV(TOIE1);`. Not sure what `_BV(TOIE1)` is. We will figure it out later. We also have `sei();` to set interrupts. Interrupts are how TVout generates video.

FINALLY we come to something interesting. We have a comment saying we are about to see how a line is rendered! Check out this cool one liner: `ISR(TIMER1_OVF_vect) { hbi_hook(); line_handler(); }`

`ISR` means this is an `Interrupt Service Routine`. As we have learned from various resources, ISR's need to be VERY short and VERY fast functions. You can see that it puts our custom function described by `hbi_hook` in, then renders the horizontal line with `line_handler`. This ISR is done for ever horizontal line.

Next we have `blank_line`. It has an if/else which, if this is `start_render`, then set `renderLine` to 0, otherwise do `vsync_line`. After this, it simply increments `display.scanLine`.

Next we have `active_line`. it ensures cycle accurate timing. the `render_line` function is just a function pointer to one of the render_line6c, 4c, etc.

Next, `vsync_line` manipulates the output compare time `OCR1A` to the vsync pulse length. At the end of the vsync lines, it restores `OCR1A` to the horizontal sync time. all of these things increment `display.scanLine`.

### Wait_until();

Now we have our very first instance of `asm`, assembly language. `wait_until` is called by `active_line` to wait for any avr instructions to finish. Lets just make sure that we first remember the syntax for assembly language:

```
__asm__ __volatile__ ("code" : OutputOperands : InputOperands : Clobbers )
```

To deconstruct assembly, make sure to always have handy the [AVR Instruction Set](http://www.mmajunke.de/doc0856.pdf). With this as reference, here is my annotation of the ASM used inside of `wait_until`:

```
__asm__ __volatile__ (
    "subi	%[time], 10\n"              //subtracts 10 from %[time] and stores the result in %[time]
    "sub	%[time], %[tcnt1l]\n\t"     //subtracts %[tcnt1l] from %[time] and store it in %[time]
"100:\n\t"                              //define 100
    "subi	%[time], 3\n\t"             //subtracts 3 from %[time] and stores the result in %[time]
    "brcc	100b\n\t"                   //brcc (Branch If Carry Cleared) 100b
    "subi	%[time], 0-3\n\t"           //subtracts 0-3 from %[time] and stores the result in %[time]
    "breq	101f\n\t"                   //breq (Branch If Equal) 101f
    "dec	%[time]\n\t"                //decrement %[time]
    "breq	102f\n\t"                   //breq (Branch If Equal) 102f
    "rjmp	102f\n"                     //rjmp (Relative Jump) 102f
"101:\n\t"                              //define 101
    "nop\n"                             //do nothing
"102:\n"                                //define 102
    : //no output operands
    : [time] "a" (time), //input operands are time and TCNT1L.
    [tcnt1l] "a" (TCNT1L) 
	);

```

Even knowing what the assembly language means literally, it is difficult to understand what is actually going on here. Lets refer to Miles' explanation on what is going on:

```
wait_until is called by active_line to wait for any avr instructions to finish
it always returns at a set time into the video line.
```

so, i need a cigarette and then im gonna take a nap ill get back to this































