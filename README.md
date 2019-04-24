# CS356 Lab 3 and 4: Polling I/O System
###### Jordan Acker, Zhanpeng Wang, Kadan Lottick, Elizabeth Chan 

Processor supports specialized instruction i/o with opcode 0x25[r][s] for getchar_ord and 0x24[r][s] for putchar_ord, where r is the register to pull into or print from, and s is the keyboard or screen to pull from or print to.

Code prints a greeting message to each terminal (per Dave's lab instructions), then enters a polling loop. In this loop it performs a computation during which typing may occur; then it pulls any characters typed on keyboard 0 and immediately prints them to screen 0; then it pulls characters typed on keyboard 1 up to a newline, and prints them first to keyboard 1 with all letters uppercase, and then to keyboard 0 with all non letters/numbers converted to underscores (per Dave's lab instructions). If no newline was typed the characters are stored, to be printed the next time a newline is typed on that keyboard.

Interrupt:
We have implemented the interrupt mechanism for our processor and the Input/Output. The implementation is that we have two program ROMs that store the opcodes for keyboard1
and keyboard2 respectively. When we type into the keyboards, an interrupt will be triggered such that the processor will execute the opcodes in the respective program ROM.
Once it finishes executing the opcodes, the processor will go back to execute the opcodes in the Main Program ROM. When we type in keyboard1, it triggers an interrupt and
immediately prints to Screen 1. If we type in keyboard2, it triggers an interrupt and stores the characters into memory buffer until an \n is entered into keyboard2. When
we type \n in keyboard2, the processor will print out the characters in the memory buffer to the Screen 2. 
