A web application was given, and since this is a crypto challenge the interesting part is:

```python
SECRET_KEY = secrets.token_hex(8).encode()

def generate_access_key(filename):
    if isinstance(filename, str):
        filename = filename.encode()
    return sha1(SECRET_KEY + filename).hexdigest()
```

After further research I discovered that this is vulnerable to a length extension Attack. A very helpful video that explains how it works: https://www.youtube.com/watch?v=H_bvdhPMizE

But here's how the attack works in short:

SHA1 processes data in 64-byte chunks, where each chunk is hashed and the result becomes the input for the next chunk. Thus, I can create a payload that holds some data (a.txt) and then manually add the same padding SHA1 would normally apply, and then (in a new block) append the new data (/../..flag.txt). 

By doing this the initialization vector for the second block is known. I can get the hash of a.txt alone from the web app and the calculation of the new hash is possible, with a known hash I could read the flag. I used `hash_extender` a tool that calculates the padding and the new hash for you:

```bash
./hash_extender --format=sha1 --data="a.txt" --secret=16 --signature=c7ef8f8165574cd870f37cad7fe12a9309bbe274 --append="/../../flag.txt" --out-data-format=html-pure
```

An important note is that the secret is 16 bytes long althought at first glance in the code it looks like its 8 bytes. I used the option `--out-date-format=html-pure`, which basically url encodes the whole data and I put the output in the browser to get the flag.

`dach2025{cybercybercybercybercybercybercyber_lypuimlb5hll5g1w}`
