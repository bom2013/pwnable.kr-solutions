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
