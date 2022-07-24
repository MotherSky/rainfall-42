when we try to execute the program without any arguments,it segfaults.
we then try to execute with a random argument, all it does is output "No!"
when we run our program with gdb we see that one of the first instructions calls atoi
and compare the eax register with the values 0x1a7 (423)
this time we give 423 as argument and we see the stdin input and what's looking like a shell
since the program has the suid bit set then our shell will execute commands as level1