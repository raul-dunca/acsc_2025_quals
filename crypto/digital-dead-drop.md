The given application contains a subtle vulnerability: it is susceptible to a padding oracle attack. [Here](https://www.nccgroup.com/us/research-blog/cryptopals-exploiting-cbc-padding-oracles/) is a very detailed explanation of how the attack works. In summary, since the application returns an error when the PKCS#7 padding is incorrect, and due to the way AES in CBC mode operates, it's possible to decrypt messages without actually knowing the key. First, let's try to find p2 (plaintext block 2):

<img src="https://github.com/raul-dunca/acsc_2025_quals/blob/main/.assets/digital-dead-drop.png">

It is known that the encrypted message (the token) is 12 bytes. So the second block c2 has 4 bytes of data, and 4 bytes are 0x04 (from PKCS#7 padding). I will denote the decrypted ciphertext as x. So in this case, I can find x[4..7] by calculating `x[i]=c1[i]^0x04`. Next I can try to modify c1 so it uses a padding of 5, it will look something like `abc55555`, where abc are 3 arbitrary bytes. Thus, it is possible to find x[3] by first calculating the correct c1[4…7] where:

```txt
c1[i]=x[i]^0x05
```

And then I can try all possible bytes values for c1[3] and only for 1 value the padding will be correct (there exists 1 value for c1[3] such that c1[3]^x[3]=0x05). Now it is known that x[3]=c1[3]^0x05. The same logic can be applied further for the rest of x. Once x is calculated, the plaintext (p2) can be computed as: 

```txt
p2[i]=x[i]^c1[i]
```

For the first block, the same logic can be applied, the only differences are that the (initialization vector) iv must be used instead of c1, all 8 bytes must be calculated since c1 has no padding and that you must not send c2 in the code when checking the padding, otherwise the padding won't affect the first block.

Here is my final solution script:

```python
from pwn import *

#context.log_level = 'debug'

HOST="port.dyn.acsc.land"
PORT=<port>


p = remote(HOST, PORT)

def get_flag_token_public_and_message_id():                 # get the flag_public_token and the message_id
    banner = p.recvuntil(b"Actions:").decode()

    for line in banner.splitlines():
        if "Validate with" in line:
            flag_token_public = line.split("Validate with")[1].strip()
            break

    p.sendline(b"3")
    messages_output = p.recvuntil(b"Actions:").decode()

    message_ids = []
    for line in messages_output.splitlines():
        line = line.strip()
        if line.startswith("- "):
            message_ids.append(line[2:].strip())

    return flag_token_public, message_ids[0] if message_ids else None

flag_token_public, message_id=get_flag_token_public_and_message_id()


def checkToken(token):                              #check using option 4 if the padding is correct
    p.send(b'4\n')
    p.recvuntil(b"Enter message ID: ")
    p.send(message_id.encode() + b'\n')
    p.recvuntil(b"Enter token: ")
    p.send(token.encode() + b'\n')

    resp=p.recvuntil(b"Actions:")

    return resp


def get_p1(iv,c1):
    iv_modified=bytearray(iv)
    padding=1
    checker=7
    current_sol=[0,0,0,0,0,0,0,0]
    while padding<=8:
        for byte_val in range(256):
            for i in range(checker,8):                          #calculate the correct padding of the previously found bytes
                iv_modified[i] = current_sol[i]^padding

            iv_modified[-padding] = byte_val
            send=iv_modified.hex()+c1.hex()                     #dont send c2 !
            resp=checkToken(send)

            if b"Token is invalid." in resp:
                current_sol[-padding]=byte_val^padding          #calculate x[i] starting from last byte where x[i]=iv[i]^p1[i]
                print(current_sol)
                break
        padding+=1
        checker-=1
    
    print("Final solution P1: ")
    print(current_sol)
    return current_sol


def get_p2(iv,c1,c2):
    c1_modified=bytearray(c1)
    current_sol=[0,0,0,0,0,0,0,0]

    for i in range(4,8):                                    #the padding for c2 is 4 since token is 12 bytes so x[4...7] can be calculated like x[i]=c1[i]^0x04
        current_sol[i]=c1[i]^4

    print(current_sol)

    checker=4
    padding=5
    while padding<=8:
        for byte_val in range(256):
            for i in range(checker,8):
                c1_modified[i] = current_sol[i]^padding    #calculate the correct padding of the previously found bytes

            c1_modified[-padding] = byte_val
            send=iv.hex()+c1_modified.hex()+c2.hex()
            resp=checkToken(send)
    
            if b"Token is invalid." in resp:
                current_sol[-padding]=byte_val^padding     #calculate x[i] starting from the 5th byte where x[i]=c1[i]^p2[i]
                print(current_sol)
                break
        padding+=1
        checker-=1
        
    print("Final solution P2: ")
    print(current_sol)
    return current_sol


token_public=bytes.fromhex(flag_token_public)
iv = token_public[:8]
ct = token_public[8:]

c1=ct[:8]
c2=ct[8:]

partial1=get_p1(iv,c1)
partial2=get_p2(iv,c1,c2)

p1 = "".join(chr(partial1[i] ^ iv[i]) for i in range(8))
p2 = "".join(chr(partial2[i] ^ c1[i]) for i in range(8))


print(f"Decrypted token is {p1+p2}")

p.close()
```

`dach2025{p4dd3d_d34d_dr0p_zbuot4f6n1ovd58j}`
