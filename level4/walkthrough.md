this level is almost exactly the same as the previous level, when we run the program with ltrace, it calls fgets with a buffer of 512 then printf that buffer.

when inspecting further with gdb we confirm that it's almost the same logic as the previous exercise, only this time we have to overwrite the value of m with a much bigger value (0x1025544 / 16930116 in int) to pass the cmp tst in line <+59>, we can use the same method as the previous exercise but we will miss a small detail. In the previous level, we got away with the padding (A*12 to fill the 64bytes printed, and change the value of m to exactly 64), if we try to do the same and pad with ~16930116 it will not go through since our buffer is only ~512 chars long and that means that the printf will only evaluate the first 512 chars.

We can take advtange of printf's width to print as many bytes, we test with the same python inline used in the last level + some modifications :
first we change the address and remove the padding since we won't be needing it here and keep adding %x until we can print our address (8049810) and that's 14 %x:
(python -c 'print "A"*8 + "\x10\x98\x04\x08" + "%x "*14' > test), we know that our address 8049810 is address 14 after printf and we can now access it using printf's $:
(python -c 'print "A"*8 + "\x10\x98\x04\x08" + "%14$x"' > test), now it will be easy to modify as we use this index + $ with %n, (python -c 'print "A"*8 + "\x10\x98\x04\x08" + "%14$n"' > test) we run gdb with the test file to see how many bytes we wrote to our address and that is 0x0c (12bytes) now we will use printf's width to print the remaining 16930104 bytes and so our final python inline will be :
python -c 'print "A"*8 + "\x10\x98\x04\x08" + "%016930104x"  + "%14$n"' | ./level4
. now that will pass the cmp test and will call system("/bin/cat /home/user/level5/.pass")


https://codearcana.com/posts/2013/05/02/introduction-to-format-string-exploits.html