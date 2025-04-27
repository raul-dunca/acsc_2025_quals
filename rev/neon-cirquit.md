First, I ran the following command to extract the content of the AppImage:

```bash
7z x neon-cirquit-1.0.0.AppImage
```

I figured out this was an Electron app and after some research, I discovered [this post](https://cypelf.fr/articles/protonic-vault/) that helped me a lot.
In this case `app.asar` was under the `resources` directory. And running:

```bash
npx asar extract app.asar app
```

I was able to recover the source files. In `index.html` I saw that the script `renderer.js` has the `checkPassword` logic. However, the password is not directly stored in code, but it uses:

```js
window.nativeAddon.checkPassword(userInput);
```

Looking at `preload.js` I noticed:

```js
const nativeAddon = require('./native-addon/build/Release/addon.node'); 
```

Thus, going to this path `addon.node` was a binary that I decompiled using [dogbolt](https://dogbolt.org/). There I saw the `CheckPassword` function, which is quite big but the most important part is near the end and is quite simple (I noticed the bytes and just thought that maybe this was the password or the flag):

```c
__builtin_memcpy(rax_42, "\x9b\x9e\x9c\x97\xcd\xcf\xcd\xca\x84\x88\xcc\xa0\x9e\x8d\xcc\xa0\xb1\xcc\xa7\xaa\xac\xa0\xcb\xcd\xa0\xce\xcc\xcc\xcb\x82", 0x1e);
        do
        {
            *rax_42 = ~*rax_42;
            rax_42[1] = ~rax_42[1];
            rax_42 = &rax_42[2];
        } while (s_7 != rax_42);
```

Basically, it just negates each byte, so I created a script to decode the password:


```python
encoded = bytearray([
    0x9b, 0x9e, 0x9c, 0x97, 0xcd, 0xcf, 0xcd, 0xca, 0x84, 0x88, 0xcc, 0xa0,
    0x9e, 0x8d, 0xcc, 0xa0, 0xb1, 0xcc, 0xa7, 0xaa, 0xac, 0xa0, 0xcb, 0xcd,
    0xa0, 0xce, 0xcc, 0xcc, 0xcb, 0x82
])

decoded = "".join(chr(~b & 0xFF) for b in encoded)
print(decoded)
```

`dach2025{w3_ar3_N3XUS_42_1334}`
