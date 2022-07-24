we run the program with ltrace and see that it call gets then puts the input to stdout then strdup.

we disassemble the program with gdb, main here isn't interesting as it only calls the p function. p declares a variable
on the stack to use it asargument for gets, we run it with the same testing string to see where the oveflow happens exactly (80chars).

we can try to overwrite the return address with a shellcode but it won't work because of an important condition in function p(), line <+39> and <+44>.
It adds the ret address with 0xb00000000 and then compares it to 0xb00000000 itself, if it the additions equals 0xb00000000
the program prints the ret address and then exits, which means the program detects and exits if we try to overwrite the return
with any address that starts with 0xb making it useless to try to execute any code on the stack.

maybe let's try executing shellcode on the heap? there is a strdup with the stack variable as argument. we can instead write some
shellcode on the buffer, overflow it then overwrite the return function to heap memory (0x804a008 aka the return of strdup) to execute our shellcode on the heap, we use this command :
(python -c 'print "\x31\xc9\xf7\xe1\xb0\x0b\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80" + "A"*59 + "\x08\xa0\x04\x08"'; cat) | ./level2
the first string is our shellcode (21 bytes) + any 59 characters to pad and make the overflow happen then the return of strdup address that we get from ltrace.

http://shell-storm.org/shellcode/files/shellcode-841.php

Another Solution: https://www.youtube.com/watch?v=m17mV24TgwY