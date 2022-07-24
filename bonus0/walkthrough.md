When running the program with ltrace, and providing we can get an idea of the functions used (read, strchr, strncpy, strcat). Not let's run it with gdb to inspect it line by line.

First we have 2 functions other than main, p and pp. (see source)
in line <+16> in main, we can see a call to pp() with 0xbffff6c6 as argument (this address is where we will concatenate the strings later), then in pp we call p 2times each time with an addresses in stacks, the p function prints a header " - " then reads our input in a 4096 buf (0xbfffe640), search for '\n' to terminate the string, then copies the first 20 characters of the buffer into the stack address given as first argument (0xbffff678 and 0xbffff68c). Note here that it's important here that p uses the same buffer for both reads.

After the second call to p, we strcypy both strings(technically only the first since the seconds follows it immediately), in the line <+91> we calculate the len of the last string and then we add a space at the end it, then we concatenate the second read again(our final string will be in the form "str1+str2+ +str2" and wil put in the address 0xbffff6c6). since this strcat is unprotected we will take advantage of it to overwrite the return address.

We can display the stackframe informations using info frame, so we can get the return address to overwrite: ebp at 0xbffff6f8, [[eip at 0xbffff6fc]].
we will run our program in gdb with the second string as "AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHH" to see where exactly the ret address will be overwritten. we get:

Program received signal SIGSEGV, Segmentation fault.
0x44434343 in ?? () . This is where we will redirect the program to our shellcode(after the first C). As for the shellcode we will make use of the ignored part in the read buffer to place it (after the first 20 characters). this will be our inline script.

(python -c 'print "A"*20 + "\x90"*20 + "\x31\xc9\xf7\xe1\xb0\x0b\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80"'; python -c 'print "B"*9 + "\x80\xe6\xff\xbf" + "B"*7'; cat) | ./bonus0 (and we add extas "\x90" NOP slide for extra safety).