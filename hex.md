---
title: "Hex Files"
date: 2017-09-20T12:58:44+01:00

---

## Replacing the user application code

The `arm-none-eabi-objdump` command is used to investigate the .hex files.

```bash
$ arm-none-eabi-objdump -x microbit-shapes.hex

microbit-shapes.hex:     file format ihex
microbit-shapes.hex
architecture: UNKNOWN!, flags 0x00000000:

start address 0x0003c0c1

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .sec1         000007c0  00000000  00000000  00000011  2**0
                  CONTENTS, ALLOC, LOAD
  1 .sec2         0000f000  00001000  00001000  000015dd  2**0
                  CONTENTS, ALLOC, LOAD
  2 .sec3         00006918  00010000  00010000  0002b8ee  2**0
                  CONTENTS, ALLOC, LOAD
  3 .sec4         00008000  00018000  00018000  0003e088  2**0
                  CONTENTS, ALLOC, LOAD
  4 .sec5         00010000  00020000  00020000  00054899  2**0
                  CONTENTS, ALLOC, LOAD
  5 .sec6         00000f80  00030000  00030000  000818aa  2**0
                  CONTENTS, ALLOC, LOAD
  6 .sec7         00003874  0003c000  0003c000  00084453  2**0
                  CONTENTS, ALLOC, LOAD
  7 .sec8         00000020  0003fc00  0003fc00  0008e323  2**0
                  CONTENTS, ALLOC, LOAD
  8 .sec9         00000004  10001014  10001014  0008e38e  2**0
                  CONTENTS, ALLOC, LOAD
SYMBOL TABLE:
no symbols
```

Comparing this to an alternative hex file shows that the binaries are the same size up until .sec6. To test this the binary is edited to replace everything after the `0x000818aa` offset with everything following the `0x000818aa` offset of another file.

The [`xxd` command](https://wiki.christophchamp.com/index.php?title=Xxd) is used to create a hex dump to edit.

```bash
$ cat microbit-shapes.hex | xdd > hexdump
```

Resulting in a file such as this:


```bash
00000000: 3a30 3230 3030 3030 3430 3030 3046 410d  :020000040000FA.
00000010: 0a3a 3130 3030 3030 3030 4330 3037 3030  .:10000000C00700
00000020: 3030 4431 3036 3030 3030 4431 3030 3030  00D1060000D10000
00000030: 3030 4231 3036 3030 3030 4341 0d0a 3a31  00B1060000CA..:1
00000040: 3030 3031 3030 3030 3030 3030 3030 3030  0001000000000000
00000050: 3030 3030 3030 3030 3030 3030 3030 3030  0000000000000000
00000060: 3030 3030 3030 3045 300d 0a3a 3130 3030  0000000E0..:1000
00000070: 3230 3030 3030 3030 3030 3030 3030 3030  2000000000000000
00000080: 3030 3030 3030 3030 3030 3030 3531 3037  0000000000005107
00000090: 3030 3030 3738 0d0a 3a31 3030 3033 3030  000078..:1000300
000000a0: 3030 3030 3030 3030 3030 3030 3030 3030  0000000000000000
000000b0: 3044 4230 3030 3030 3045 3530 3030 3030  0DB000000E500000
000000c0: 3030 300d 0a3a 3130 3030 3430 3030 4546  000..:10004000EF
000000d0: 3030 3030 3030 4639 3030 3030 3030 3033  000000F900000003
000000e0: 3031 3030 3030 3044 3031 3030 3030 4236  0100000D010000B6
000000f0: 0d0a 3a31 3030 3035 3030 3031 3730 3130  ..:1000500017010
```

This is also done for a second hex file containing another program. Then remove everything after the `0x000818aa` offset for the first file and replace it with everything after the offset in the second file.

```bash
0081880: 3030 3030 3030 3030 3030 3030 3030 3030  0000000000000000
00081890: 3030 3030 3030 310d 0a3a 3032 3030 3030  0000001..:020000
000818a0: 3034 3030 3033 4637 0d0a 3a31 3030 3030  040003F7..:10000
000818b0: 3030 3037 3038 4533 4239 3243 3631 3541  000708E3B92C615A
000818c0: 3834 3143 3439 3836 3643 3937 3545 4535  841C49866C975EE5
```

The hex dump can then be reverted using the `xxd` command.

```bash
$ xxd -r hexdump > microbit-frankenstein.hex
```

This can now be flashed to the device and the second programs code will execute!

