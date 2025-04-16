During my research about `Nokia SCKL` I found [this](https://www.smssolutions.net/tutorials/smart/sckl/). Given the SCKL string, I found out that the messages represent the sending of a group logo. This is important because now I also know the size. A group logo must be 72x14 pixels. Cool, now I wrote a script to reconstruct the picture and read the flag:

```python
from PIL import Image

data_lines = [
"0B05041583000000030103013000480E01FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFBFFF9FFFFF3FFC71FFFFFFDFFFEFBFFDED0F32","0B050415830000000301030273DFFCC18FF8EDB7BDE1DFFB6FB7FB71B7BAEFDFFB6DB7FB7B8F12718FFCF30FF8E3BFFFFFFFFFFFFFFFFF1FFFFFFFFFFFFFFFFF","0B0504158300000003010303FFFFFFFC0FFFFC0FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF00"]

def extract_payload(line):
    return line[24:]        #tried different offsets, from theory I thought I needed [28:]

payload_hex = ''.join([extract_payload(line) for line in data_lines])

# skip the first 5 bytes, which usually contain header data in payload
bitmap_hex_data = payload_hex[10:]
bitmap_bin_data = bin(int(bitmap_hex_data, 16))[2:]

width_pixels = 72
height_pixels = 14
expected_bits = width_pixels * height_pixels

actual_bits = len(bitmap_bin_data)
if actual_bits < expected_bits:
    bitmap_bin_data = bitmap_bin_data.zfill(expected_bits)
elif actual_bits > expected_bits:
    bitmap_bin_data = bitmap_bin_data[:expected_bits]

img = Image.new('1', (width_pixels, height_pixels))
pixels = img.load()

for y in range(height_pixels):
    for x in range(width_pixels):
        bit_index = y * width_pixels + x
        bit_value = int(bitmap_bin_data[bit_index])
        color = 0 if bit_value == 1 else 1
        pixels[x, y] = color

img.save('flag.png')
```

I start by removing the header data, then I convert and combine the three messages into a single binary stream. Since I know the size of the image, I simply color each pixel black for a 1 and white for a 0. This returns: 

<img src="https://github.com/raul-dunca/acsc_2025_quals/blob/main/.assets/nokia.png">

`dach2025{pixel_otb_69}`
