we execute the program and it's a simple stdin that does nothing
after running the program with ltrace we see that all it does is call the gets function then exits
the gets function has known security errors because of the inability for the program to determine the length of input
which is susceptible to a buffer overflow attack.

we run it with gdb to inspect for any hints, disassembling the main function using "disass main" gives us a simple program
that declare a variable on the stack of length[0x50] that's used as argument for gets. that's all.
now we check if there is any hidden function using "info func", apart from regular system functions we see a run function
this functions writes a message to stdout then calls the system function with "/bin/sh";

we then executes a buffer overflow attack with the help of gets to execute the run() function;
we run then enter the following string "AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWYYYYZZZZ"
to see where the oveflow happens exactly.

since our input exeeds the length declared before (0x50), our program segfaults trying to execute the "0x54545454" (TTTT) instruction.
we now know where to put the address of run function to be executed instead of "0x54545454", we examine run "x run" to get its address 0x8048444. 
all we have to do now is concat it to our string in the position of "TTTT", we use the following inline python script:
python -c 'print "AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSS" + "\x44\x84\x04\x08"' > test
(here the address of run is flipped to little endian representation). we pipe the content of test to our program, run is executed successfully
but we don't see the shell, that is because it's closed immediatly, we can chain it with cat to resolve this;
(cat test; cat) | ./level1
