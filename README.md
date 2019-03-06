# CS356Lab3
A github repo for CS356 Lab3


# Lab 3: Co-Design of an I/O System
###### Jordan Acker, Zhanpeng Wang, Kadan Lottick, Elizabeth Chan 

## In this lab, you will work as part of a team, to extend the HERA hardware to include connections to command-line terminals, with software to demonstrate input and output of characters. It will give you a chance to explore the collaboration features of git, as well as work on a Co-Design project and learn about hardware and operating systems.

### Dave's plans/suggestions for next year:
Require git to exist and be usable before any work is actually done (except for the design document)
Require polling be submitted first, perhaps a week or two before interrupts are due

### The requirements for the operation of the system are:
- The HERA system must include at least two keyboards and two screens, in addition to the CPU, instruction ROM, and data RAM from CMSC 240. The keyboards, screens, ROM, and RAM should be visible when viewing the "top level" circuit, i.e., without opening any modules.
- It must be possible for a HERA program to call a function putchar_ord to print one character on one screen (the function should take two parameters, the first of which identifies which screen, and the second of which is the ASCII code of the character to be printed … you are only responsible for the characters that Logisim handles correctly). To avoid conflicts with the standard library for HERA, put the following line in your HERA program before any #include of that library or its data:
    ```
    #define  WROTE_CS356_IO 1
    ```

- It must be possible for a HERA program to call a function getchar_ord to read a character from either keyboard (this function should take a parameter identifying which keyboard is to be used, and should return the ASCII code of the character entered), and also to have some way to determine whether input is ready (either some other function, or some special option for getchar_ord, or whatever you like; but note that the usual behavior of getchar_ord is not to return until the user has provided a character). To avoid conflicts with the standard library for HERA, put the following line in your HERA program before any #include of that library or its data:
    ```
    #define  WROTE_CS356_IO 1
    ```
- Your system should work for any correct HERA program that uses calls to getchar_ord and putchar_ord to do input and output, and, in particular, for demo purposes include the test program described in the next bullet item. Additionally, any legal HERA program that does not do I/O (or try to store data in the addresses used for memory-mapped I/O, if you choose that option) must still function as it would have without you modifications for this lab (so, for example, you can't decide not to have an "ADD" instruction any more, or just to have 4 registers … don't break existing features).
- You should provide a demonstration test program that does the following:
  - The program first prints greetings to both terminals, with "hi" going to terminal 1 and something else ("hello", or a brief greeting in some other language) going to terminal 2. The H and the i should be placed in a register with SET (or SETLO); the characters from the second greeting should be extracted from a string defined with a DLABEL/LP_STRING definition in the data segment. When you're developing/debugging output, you may want to have a HALT( ) right after these to steps, so you can get output working before attempting input.
  - The program then does some computation that takes a few seconds. Possibly the user types something at this point.
  - The program then reads characters from keyboard 1 and prints them immediately to screen 1, until it gets a newline or return.
  - The program then reads characters from keyboard 2, without immediately showing them, until the input character is a newline or linefeed. After reading the newline/linefeed, it prints the entire line to screen 2, but with all lower-case letters converted to upper-case, and prints the original line, with all characters that aren't numerals or letters replaced by underscores, to screen 1.
  - The program then enters a loop in which it checks each keyboard to see if it has a character, and if so, prints that  character to both screens, but with all letters from keyboard 1 showing up in upper-case on both terminals, and all letters from keyboard 2 showing up in lower-case on both terminals.
  - See this [Python test program](https://docs.google.com/document/d/1GwSVHJzAqS8MlTPDZCcd7VHrWiF4_Jt_5j-yH3Z3Bn8) for an example of what's needed for the demo 


### For full credit (see below), the system must also do the following (interrupt-based I/O is the way you can ensure these):
- It must not "lose" characters from either keyboard. In other words, if the test program above is run and keystrokes are entered at keyboard 2 before the first newline is typed on keyboard 1, those keystrokes should be included in the screen 2 output (you may assume that the user will not get ahead of the interrupt service routine by more than the hardware buffer size, i.e., 16 characters, and you may also assume that they won't get ahead of the overall progress of the program by more than some fixed but larger size, such as 128 or 2048 characters)
- It must be able to perform calculations without spending time checking keyboards except when a character has actually been typed (you can modify the demo/test described above to perform some calculation between the initial greeting and the part with I/O, e.g. it could either just keep incrementing some register, preferably one that's visible on the main screen, or compute something more interesting such as searching for numbers n, a, b, and c, such that a^n + b^n = c^n modulo 2^16 (i.e., in single-precision arithmetic), up to some limit that will force it to run for 30 seconds or so of actual "wall clock time" (i.e., time for us to enter input at the keyboards, or it could compute some number in the fibonacci sequence in the slow recursive way, for some example that takes about 30 seconds or so).

### System design options and recommendations:
- You may use either memory-mapped or specialized-instruction I/O. Note that neither can be easily tested in HERA-C, but both can be included in your HERA program and processed with the HERA assembler:
   - If you have a specialized instruction, express it with the "OPCODE" pseudo-instruction (mentioned in Section 3.1 of the HERA booklet). Just as Hassem turns "ADD(R1, R1,R4)" into 0xA114 for Figure 4.1 of the HERA booklet, and turns "BGER(SKIP_NEGATION)" into 0x0302 for Figure 5.1, it would turn OPCODE(0x3211) into 0x3211. Note that the HERA-C simulator can't run the OPCODE instruction, but you can assemble those parts and then use the HERA processor from your hardware team to test them.
   - If you use memory-mapped I/O, you can do LOAD and STORE operations in HERA-C, but they will just act like regular memory operations. Please choose addresses well above 0xc000 (where the heap starts), for compatibility with various memory-allocation conventions used by the HERA software ecosystem.
   - You can test more complicated code that relies on your I/O by using the Tiger library I/O operations when running in HERA-C and on the actual microprocessor, by using the preprocessor directives "#if defined HERA_C" and  "#if ! defined HERA_C". If you put one of those above some lines, and  "#endif" after those lines, they'll be used only in HERA_C or only in Hassem.
- For partial credit, or as a starting point toward a full-credit design, you may implement polled input, but I think you'll need to build an interrupt-driven system to meet the full-credit input specification above.


### I strongly recommend you approach this in a step-by-step fashion:
- Design and implement output first: in any order,
   - Meet with the other hardware or software team(s) with whom you're collaborating, and discuss the overall design of the output system. Note that, due to the degree of simplification in logisim, you may not have to deal with issues of polling, interrupts, or buffering that could arise with a real terminal, but that's fine for this course because it gives a simpler warm-up step.
   - Create one or two documents in your team's folder within the Co-Design Teams and Shared Documents for Lab 3 folder, to describe your overall shared designs for your output and input systems: will you use memory-mapped or specialized-instruction I/O, and exactly what memory addresses or instructions will be used? You are welcome to use Google's access controls to ensure that only your team members can edit the documents, but please make sure that all members of your counterpart team, and the instructor and T.A.'s can all at least comment on these documents.
   - Create git repositories for (each of) your hardware and software teams. You can use the techniques described at the end of Lab 2, but include --shared=group at the end of the line git init --bare, so that all group members will have access. For the step in which you "copy files from the renamed version", someone from hardware team can just copy their .circ file into the newly-cloned empty repository, and someone from the software team can copy from an existing HERA project (e.g., HERA_Individual_Project), and then do a commit and Push (either from Eclipse, the command line, or by typing "git gui" and using the graphical interface to click on files in the upper left pane, enter a commit comment in the lower right, and then use the Commit and then Push buttons.

*** IN FUTURE YEARS, DAVE/JD SHOULD SEE IF THE CHOWN CAN BE DONE BY THE STUDENTS, AND ADD INSTRUCTIONS HERE SO AS NOT TO ADD  DELAY; ALSO, USE 'ls' TO CONFIRM IT IS PROPERLY BARE ***  ALSO NEED TO BE SURE UMASK IS OK

Once you have created this repository, send email with its name to davew@cs.haverford.edu and jcammisa@haverford.edu so that we can ensure it is associated with your team's group. After we set that up, your teammates should be able to use "git clone" as follows (with the userWhoCreatedRepo in my example replaced by the id of the user who created the repository, and with RepositoryName replaced by the repository name, of course):
```
  cd ~/cs356
  git clone /home/courses/students/userWhoCreatedRepo/git-cs356/RepositoryName
  # Note: you can do this on e.g. your laptop,
  #        if you put "yourId@church.cs.haverford.edu:"

  # at this point, you should be able to connect as if to lab starter files
```
- Have your hardware and software teams meet to test the combination of hardware and software; debug and (if necessary) update your design, and get the system working before moving on to address input. Your goal here should be to implement putchar_ord and also write a main demo/test program that calls it once or twice for specific values you've put in registers, and then has a loop to print all characters in a string (defined via LP_STRING).

- After you have output working, design and implement the input system; if you are feeling ambitious, you are allowed to go straight to an interrupt-based system, but I recommend you start with a polling-based system, commit a partial-credit result with some very specific comment such as "polling approach working now", and then revise your design and implementation to make use of interrupts.

### The requirements for performing this lab are as follows:
- Form a team of 2-4 students who agree to work together on Hardware or Software. One important attribute of a good team is a set of schedules that are similar enough to allow work at the same time. Note: your professor may instead choose to assign teams rather than have students self-organize.
- Work together with the collaborating team(s) to produce a Co-Design Specification that identifies your basic approach (see above) and all the details needed to defined the responsibilities of the hardware and software teams, and details needed for a full implementation (i.e., if you have a special I/O instructions, what is the operation code, and what determines which device you're using? if you use a memory-mapped I/O, what addresses correspond to what operations on what devices?). All teams must come to consensus on this specification and any changes to it.
- Note that your professor will play two roles:
   - As customer, will only be satisfied (and grade for 100% on the "system works" points for either team) if the combined system actually works (send email with the subject line "Message for Lab 3 customer" to communicate about any issues relevant to your customer).
   - As "upper-management", can serve as a resource if you are having communication or collaboration issues within or between teams (send email with the subject line "Message for Lab 3 senior management" to communicate about any issues relevant to senior management, e.g. if there is some reason the teams can't work smoothly together. Note that upper management does not want to become involved in the details of problems, but will do so if necessary).
- Implement your part of the design (i.e., the hardware or software), and, as needed meet with the other teams to clarify/update your specification. All teams must agree to updates to the specification, as well as the original document. Each team member must actually do some "hands-on" work on some element(s) of the system implementation (as well as understand all elements of their team's contribution and the co-design specification).
- Together with each collaborating team, test your complete system, debug as necessary, and update the specification if needed (only after communication and agreement with any other team(s) involved).
- Ensure that you and all other members of your team have a detailed understanding of your part of the system and the shared specification … your grade will depend primarily on the completeness and correctness of your working system, and also in part on your ability to describe the system and specification, and on the ability of your teammates to describe these things.

### Grading details:
- 90% for the design and functionality of your complete system:
   - 20% for design description, coherence of design and implementation, and style
     - 10% for having a coherent, workable design described in your shared document
     - 10% for having your implementation actually fit with that design (regardless of whether the implementation works)
     - points may be subtracted from the above for egregiously poor style, including (but not limited to):
       - In the HERA circuit, all keyboards, screens, ROM, and RAM must visible in top-level circuit, rather than inside a module (they may make sense in a module, but having them there really complicates using/testing the system, since we test by loading the memory chips and then typing and watching the screens).
       - Unnecessary/inappropriate files in the git repo, especially if these files are numerous or large, potentially slowing down git commands and creating confusion.
     - 30% for having hardware and software that work (for your test and ours) for putchar_ord and for looping over a string and outputting each character
     - 40% for working input, with credit awarded for the maximum of
       - up to 20% for getting a polling-based input system working
up to 40% for getting an input system that meets the "full credit" definition above
       - 5% for your ability to demonstrate your understanding during the individual lab review/grading session
       - 5% for the overall ability of your teammates to demonstrate an understanding of your system
- There are some extra credit options that are modelled on my thoughts about things that are educationally valuable and also would reflect a real-world advantage for a company producing a product. There are several options, no individual person can add more than 5% total extra credit:
   - Extra technical functionality can help your system succeed, so there are bonuses if your team implements the following and you can explain it:
     - 3% for the Unix echo and echoe options of stty (in which each keyboard is associated with a specific screen, and as each character is typed it appears on the screen during the interrupt processing, rather than later, with backspace being processed then too), with function(s) to turn these options on and off individually.
     - 3% for flexible input buffering, i.e., someone could type a 280-character line and a 50-character line and a 300-character line before the program requested any input, and have the interrupt routine buffer all of that, without having allocated all that storage before the input appeared.
- Extra team expertise can help your system to succeed, and it is not always easy to hire perfect engineers, so there are bonuses if you can help any teammates address lingering confusion or difficulty with the prerequisite material for this class. Thus, you'll get credit for that, based on
   - 1% for helping one teammate move from inexperience on a core topic (flags and branching; load and store; call and return) to confidence. "Teammate" can be someone on your team or your collaborating hardware/software team; "inexperience" means not having done this in lab or not having gotten at least 60% of the credit for lab work on this in a previous course; "confidence" means they add a comment to their grading document stating that you helped them get to a point where they aren't worried that an exam question could assume confidence with this topic.
   - If you have been helped in this way, you neither gain nor lose credit by reporting that a specific person helped you, but you can't assign more than 1% total per topic on which you got help.
   - You can get 2% or 3% by helping 2 or 3 teammates or by helping a teammate with 2 or 3 topics, but not more than 3% total
   - If your team seems to have too many inexperienced folks vs. confident folks, don't forget that you can always ask for help from the instructor or T.A.'s (if you're worried that multiple/all experienced folks might be able to go past that 3% total, definitely let us know!)



