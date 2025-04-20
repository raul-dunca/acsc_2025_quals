A binary files is given that is designed for AVR microcontrollers. Running `strings` on the file, I noticed `atmega328p`. I installed some [helper files](https://www.jonaslieb.de/blog/arduino-ghidra-intro/) to make Ghidra support `atmega328p`. I had to disassemble the first block of code to actually see the interrupt table. The interrupt table is basically just a bunch of jump instructions. What is interesting here is the first jump, which is what happens when the board is turned on or reset. Thus, I discovered the main function called `FUN_code_0003e6()`. I also used  `simavr` so I could do some dynamic analysis with `gdb`.

```bash
simavr -m atmega328p -f 16000000 -v -t  -g GRID_WAVE_4X8_LangleyMicros.vr
```

This basically simulates an atmega328p microcontroller running at 16MHz, and opens the port 1234 to which it is possible to connect using `avr-gdb`:

```bash
avr-gdb -ex "set architecture avr" -ex "target remote :1234"
```

In the AVR architecture, the code memory (.text) and data memory (.data) are separated. I looked at the `.data` section, where I saw different strings, including the beginning of the flag format `dach2025{` but not the actual flag. I notice that this data was references in a function called `FUN_code_000081()`, which was used in the main function towards the end. I was thinking the flag will be printed by this function after previously passing some checks. After some analysis, I realized that the function `FUN_code_00006()` reads/expects some kind of input because it reads data from `0xc6`, which is the location where single characters from the 0 interface will be handled. I didn’t know how to pass data, so I used `gdb` to just modify the registers and to pass 1 character at a time. After the input is read, I thought some kind of encoding was done, and the output is saved at `0x8008d2`. After that, the function `FUN_code_000452()` seems to compare 2 pointers byte by byte the output of the encoding and some data starting at  `0x8008e6` for exactly 20 bytes. Looking in `gdb` at the data, hold by that address:

```txt
(gdb) x/20xb 0x8008e6
0x8008e6:	0xea	0xda	0x28	0x98	0x4e	0xce	0x76	0xce
0x8008ee:	0x25	0x8f	0x35	0x4d	0x1a	0x87	0xc1	0x40
0x8008f6:	0xa2	0x06	0x23	0x47
```

For now, I just modified the registers to pass the check of the 2 pointers being equal. I made them point at the same address. Finally, I looked into `FUN_code_000081()` which has some calls to `FUN_code_00005e()`, this seems to be print functions. I also notice that it tried to print `dach2025{` and then the values stored at `0x80086e` which is the location where my data was read from the input. So the flag is not printed, instead I have to find the right input to pass the comparison. After a little analysis I realized that the "encoding" takes 2 characters at a time and creates the byte equivalent:

So the correct input would be: `eada28984ece76ce258f354d1a87c140a2062347`.

`dach2025{eada28984ece76ce258f354d1a87c140a2062347}`
