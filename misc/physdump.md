Given that the Windows OS was provided in the challenge description and because a physical memory dump was included, I was quite sure I had to use volatility. Furthermore, I used [volatility3](https://github.com/volatilityfoundation/volatility3), since volatility2 supports only some memory images and `Windwos 10 22H2 ()` was not one of them. 

Running volatility3 on the given physical memory dump will not work because volatility expects the dump to start at 0x0, but as specified in the description: “The guest OS physical memory was mapped @ 0x100000”. As a result, I had to fill the memory from 0x0 to 0x100000 with null bytes:

```bash
dd if=/dev/zero bs=1 count=$((0x100000)) of=padding.bin
cat padding.bin mem_100000_2146435072.bin > full_memory.bin
```

Now I could use volatility3 and I started by listing the processes present in the given memory image:

```bash
vol -f full_memory.bin windows.pslist
```
In the output I noticed two suspicious processes with the same parent id (PPID):

<img src="https://github.com/raul-dunca/acsc_2025_quals/blob/main/.assets/physdump.png">

I then started to investigate these 2 processes by looking at their open handles, dumping the memory regions of the processes, and analyzing them, but I found nothing. I then tried to dump files from the memory of the processes:

```bash
vol -f  full_memory.bin -o "../dump_5164" windows.dumpfiles -pid 5164
vol -f  full_memory.bin -o "../dump_4808" windows.dumpfiles -pid 4808
```

And I was able to get the `cyber_binary.exe` file, which I then decompiled using Ghidra. I then checked different functions and discovered `FUN_7ff79ec11000` which XORs two vectors byte by byte. Executing the XOR operation in python reveals the flag:

```python
a = [7, 0xfe, 0xd8, 0xf6, 0xe2, 0x15, 0x4e, 0x92, 0xef, 0x16, 0x32, 0x60, 0xb1, 0x7c, 0x8e, 0x90,
     0x4b, 0x2d, 0x6b, 0x71, 0x4d, 0xdc, 0x3d, 0xca, 0xb8, 0xa9, 0xc, 0xd2, 0x3a, 0xde, 2, 0x22, 0xa3, 200,
     0x9e, 0xa7, 0x1e, 0x79, 0xe3, 0x46, 0xda, 0xd, 0x30, 0xa7, 0xe4, 0x9a, 0x2f, 0x1a, 0x89, 0xf6, 0xe9, 0xc2, 0xb3, 0x26]


b=[99,  0x9f, 0xbb, 0x9e, 0xd0, 0x25, 0x7c, 0xa7, 0x94, 0x71, 0x47, 0x53, 0xc2, 0xf, 0xd1, 0xe9, 0x7b, 0x58,
0x34, 0x1d, 0x28, 0xe8, 0x4f, 0xa4, 0xdd, 0xcd, 0x53, 0xa1, 10, 0xb3, 0x67, 0x7d, 0xd1, 0xfc, 0xe9, 0xf8, 0x6c,
0x4d, 0x94, 0x19, 0xa8, 0x39, 0x47, 0xf8, 0x82, 0xaa, 0x5d, 0x7f, 0xe7, 0x85, 0xd8, 0xa1, 0xc0, 0x5b]


flag=""
for i in range(len(a)):
    flag+=chr(a[i]^b[i])
print(flag)
```

`dach2025{gu3ss_y0u_le4rned_s0me_r4w_r4w_r4w_f0rens1cs}`
