when we first run the program with ltrace we see a call to printf("%p, %p \n") the 2 params being null. then we keep getting fgets with stdin the same printf and it looks like we're inside an infinite loop. we run the program with gdb to analyse further.
first we examine for any function or variable that could help (info func, info var), we see two variables auth(0x08049aac) and service(0x08049ab0). then we disassemble main and try to look for any functions or strings that could help :

the first function is our printf, we examine it's arguments : "%p, %p \n", 0x8049aac and 0x8049ab0, these two addresses that we try to print are the same as auth and service above.
then fgets that takes stdin and writes 32 chars to buf.
we then have multiple strings comarisons that compares buf to multiple strings(auth, reset, service, login) then act accordingly:
"auth " = mallocs 4 bytes for the variable auth and if buffer is less than 30 chars strcopy the buf to the chunk allocated (that means this strcpy is protected and we can't really overflow it).
"reset" = frees auth;
"service " = strdup to allocate then copy buf + 7 into the service variable
"login " = launch a shell if the content of 0x804a028(auth+0x20) != 0, otherwise prints "Password:\n".
Here we can try to overflow the service variable to write into 0x804a028(auth+0x20).
first we auth any char to allocate for auth. this is a representation of heap after "auth A" :
0x804a008:	0x00000a61	0x00000000	0x00000000	0x00000011
0x804a018:	0x000a6120	0x00000000	0x00000000	0x00020fe1
0x804a028:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a038:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a048:	0x00000000	0x00000000	0x00000000	0x00000000
since we know the next service strdup will be put at 0x804a018 we can use it to overflow 0x804a028 and launch the shell and that's by writing more than 16 bytes, for example:
serviceAAAAAAAAAAAAAAAA
then we login to make the check and call system.


https://www.youtube.com/watch?v=ZHghwsTRyzQ