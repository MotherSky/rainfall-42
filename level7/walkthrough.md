we run the program with ltrace, we get a segmentation fault in strcpy that got a NULL src, we run it again with an arguments, there is another segfault in a second strcpy which means we should enter 2 args.

when entering the 2 args, we see 4 consecutive calls to malloc(8), and they return continous chunks of memory from 0x0804a008 to 0x0804a048, then we see the 2 strcpys that copy the first and second arguments to 0x0804a018 and 0x0804a038 respectively. then calls fopen("/home/user/level8/.pass", "r") which returns 0.(Note : that happens mainly because ltrace and gdb run the program with the launcher permissions even if it has suid bit set for security reasons, and this is why our program will segfault in both ltrace and gdb since our uid does not have permissions to read /home/user/level8/.pass and if we run the program normally it won't segfault and will return the proper file pointer).

when we run gdb and inspect for functions and variables in the program(info func, invo var)we see a function m(0x080484f4) and a variable c(0x8049960). when we disassemble main we see that the program calls fgets with the return of fopen("/home/user/level8/.pass", "r") and put the content of the file in the c variable, while in the m function we printf the value of 0x8049960 (he c variable where we put the contents of the /home/user/level8/.pass file).

The important detail to solve this level is that 1st(0x0804a008) and 3rd(0x0804a028) malloc calls are divided in two parts:
the first part contains 0x1 and 0x2 respectively, and the seconds part contains the addresses where to strcpy argv[1] and argv[2] respctively.
here is a representation of what the memory chunks allocated looking in the heap (after both strcpy)if we run the program with AAAA BBBB:

0x804a008:	0x00000001	0x0804a018	0x00000000	0x00000011
0x804a018:	0x41414141	0x00000000	0x00000000	0x00000011
0x804a028:	0x00000002	0x0804a038	0x00000000	0x00000011
0x804a038:	0x42424242	0x00000000	0x00000000	0x00020fc1
0x804a048:	0xfbad240c	0x00000000	0x00000000	0x00000000

So by overflowing argv1 we can choose where to write, and the second strcpy will write the content of argv2 to that address.we can then solve this level with the following command:
./level7 $(python -c 'print "A"*20 + "\x28\x99\x04\x08"') $(python -c 'print "\xf4\x84\x04\x08"').
what we do here is overflow argv[1] to write the address where puts@plt jumps to (0x8049928) then we overwrite it m's address (0x080484f4).