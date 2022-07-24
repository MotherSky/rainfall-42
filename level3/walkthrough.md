we run the program with ltrace to get the input, unlike previous exercices, this one uses fgets which means the input is restricted to 512 and that means we can't overflow the buffer anymore. After it we see a call to printf with our buffer.

we disassemble with gdb, all what main does is call the v functions which calls fgets and printf, then we see a cmp operations that compares the address 0x804988c with the value 0x40(64), if the cmp is successful we print a string then call system("/bin/sh").

We have to change the address 0x804988c (variable m) to 0x40 to access the shell, we can use a format string exploit, we enter "%p %p %p or %x %x %x" as input and we see indeed that addresses from the stack are printed, so we can read memory addresses from printf. What's interesting, is that with the n modifer ("%n"), we can also write on memory since %n writes the numbers of characters written so far to the corresponding argument by its pointer. and that means we can print the address of m on the stack and then write into it using %n. but first we have to get the address of m on the stack so we can change its values later, and we can do that using printf itself.

we do this by using the following inline python :
(python -c 'print "A"*8 + "\x8c\x98\x04\x08" + "A"*12 + "%x "*5 + "%n"'; cat) | ./level3
this command does write 8 "A"s on the stack, our address then another 12 "A"s, the latter is for padding since we need to write exactly 0x40 (64) on our address to pass the cmp test, and %n writes the number of chars/bytes written to the pointer, so printf should also print 64 bytes.

https://web.ecs.syr.edu/~wedu/seed/Labs/Vulnerability/Format_String/Format_String.pdf