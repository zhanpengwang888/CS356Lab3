# CS356 Lab 3: Co-Design of an I/O System
###### Jordan Acker, Zhanpeng Wang, Kadan Lottick, Elizabeth Chan 

Processor supports specialized instruction i/o with opcode 0x25[r][s] for getchar_ord and 0x24[r][s] for putchar_ord, where r is the register to pull into or print from, and s is the keyboard or screen to pull from or print to.

Code prints a greeting message to each terminal (per Dave's lab instructions), then enters a polling loop. In this loop it performs a computation during which typing may occur; then it pulls any characters typed on keyboard 0 and immediately prints them to screen 0; then it pulls characters typed on keyboard 1 up to a newline, and prints them first to keyboard 1 with all letters uppercase, and then to keyboard 0 with all non letters/numbers converted to underscores (per Dave's lab instructions). If no newline was typed the characters are stored, to be printed the next time a newline is typed on that keyboard.



