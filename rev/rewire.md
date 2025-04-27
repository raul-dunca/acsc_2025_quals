
I used [dogbolt](https://dogbolt.org/) to decompile the binary and noticed the `disable_cctv()` function. I didn't notice a way to call this function, for example through a buffer overflow, so I just created my own python program that takes what seemed to be the encoded flag and I decoded it using the same logic as in the `disable_cctv()` function:

```python
encrypted_bytes = [
0x30, 0x19, 0x06, 0x26, 0x66, 0x48, 0x57, 0x7b, 0x2f, 0x1c, 0x51, 0x23, 0x3a, 0x27, 0x25, 0x3c, 0x67, 0x27, 0x1c, 0x7e, 0x21, 0x27, 0x04, 0x11, 0x26, 0x49, 0x15, 0x3e, 0x31, 0x0a, 0x01, 0x7e, 0x37, 0x27, 0x0a, 0x3c, 0x0b, 0x0f, 0x0d, 0x7a, 0x20, 0x59, 0x5a, 0x11, 0x62, 0x4f, 0x03, 0x2b
]
key = 0x4e657854  
decrypted_bytes = []
for i in range(len(encrypted_bytes)):
    shift = (i & 3) * 8
    key_byte = (key >> shift) & 0xFF
    decrypted_bytes.append(encrypted_bytes[i] ^ key_byte)

last_part="5JT+)"
for i in range(len(last_part)):
    shift = (i & 3) * 8
    key_byte = (key >> shift) & 0xFF
    decrypted_bytes.append(ord(last_part[i]) ^ key_byte)
print("Flag:", "".join(map(chr, decrypted_bytes)))
```

A trick done by the author is that they saved a string after the hex encoded flag, and that string is also part of the flag, since the things are stored in continues memory and in code the `for` exceeds the hex string length. You can see it in my code, that's why I have the `last_part` code.

`dach2025{d4mn_@r3_y0u_a_r1pperd0c_or_wh4t!?_67fea21e}`
