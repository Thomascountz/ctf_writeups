---
ctf: picoctf_gym
competition: false
categories: [forensics, web, cryptography, reverse engineering, binary exploitation]
tools:[file, hexed.it, stegonline, aperisolve]
url: https://play.picoctf.org/practice/challenge/205
captured: 2022-11-03
flag: picoCTF{w1z4rdry}
summary: After fixing the magic bytes of a PNG file, we can discover the flag within the red layer by adjusting the image curves or by using a layer inspection tool
---

> Ron just found his own copy of advanced potion making, but its been corrupted by some kind of spell. Help him recover it!

The file we're given has no extension and running `file` just reports `data`.

```shell
$ file advanced-potion-making
advanced-potion-making: data
```

If we look at the [magic_bytes](../../reference/magic_bytes.md) in the hexdump, we can see—what appears to be—a corrupted PNG header as evidenced by the `IHDR` (the _image header_ chunk) portion of the first 16 bytes.

```bash
$ hexdump -C -n16 advanced-potion-making
00000000  89 50 42 11 0d 0a 1a 0a  00 12 13 14 49 48 44 52  |.PB.........IHDR|
```

We can see that it's corrupted because the first 8 bytes of a PNG must be `89 50 4e 47 0d 0a 1a 0a`.

> The first byte is used to detect transmission over a 7-bit channel--for example, email transfer programs often strip the 8th bit, thus changing the PNG signature. The 2nd, 3rd, and 4th bytes simply spell ``PNG'' (in ASCII, that is). Bytes 5 and 6 are end-of-line characters for Macintosh and Unix, respectively, and the combination of the two is the standard line ending for DOS, Windows, and OS/2. Byte 7 (CTRL-Z) is the end-of-file character for DOS text files, which allows one to TYPE the PNG file under DOS-like operating systems and see only the acronym ``PNG'' preceded by one strange character, rather than page after page of gobbledygook. Byte 8 is another Unix end-of-line character.[^1]

| Hex Value | Decimal  Value | ASCII Interpretation                                         |
| --------- | -------------- | ------------------------------------------------------------ |
| 89        | 137            | A byte with its most significant bit set ("8-bit character") |
| 50        | 80             | P                                                            |
| 42        | 78             | N                                                            |
| 11        | 71             | G                                                            |
| 0d        | 13             | Carriage-return (CR) character, a.k.a. CTRL-M or ^M          |
| 0a        | 10             | Line-feed (LF) character, a.k.a. CTRL-J or ^J                |
| 1a        | 26             | CTRL-Z or ^Z                                                 |
| 0a        | 10             | Line-feed (LF) character, a.k.a. CTRL-J or ^J                |

Let's fix the header.

```bash
$ hexdump -C -n16 advanced-potion-making.png
00000000  89 50 4e 47 0d 0a 1a 0a  00 00 00 0d 49 48 44 52  |.PNG........IHDR|
```

Now we can open the image.

![](./attachments/advanced_potion_making_png.png)

To the naked eye, this looks to be a solid red image. 

- Fix png magic bytes
	- pngcheck
		- Online version: https://www.nayuki.io/page/png-file-chunk-inspector
	- https://hexed.it/
	- https://asecuritysite.com/forensics/png?file=bg.png
- LSB/layer analysis
	- https://stegonline.georgeom.net/image
	- https://www.aperisolve.com/54be76cbb51bde77f453f8c64ed407a4


```bash
$ hexdump -C advanced-potion-making.png | head
00000000  89 50 4e 47 0d 0a 1a 0a  00 00 00 0d 49 48 44 52  |.PNG........IHDR|
00000010  00 00 09 90 00 00 04 d8  08 02 00 00 00 04 2d e7  |..............-.|
00000020  78 00 00 00 01 73 52 47  42 00 ae ce 1c e9 00 00  |x....sRGB.......|
00000030  00 04 67 41 4d 41 00 00  b1 8f 0b fc 61 05 00 00  |..gAMA......a...|
00000040  00 09 70 48 59 73 00 00  16 25 00 00 16 25 01 49  |..pHYs...%...%.I|
00000050  52 24 f0 00 00 76 39 49  44 41 54 78 5e ec fd 61  |R$...v9IDATx^..a|
00000060  72 e3 4c 94 a6 59 ce 16  6a fe 76 cd fe 57 d7 dd  |r.L..Y..j.v..W..|
00000070  5b 18 45 e9 4b 8a 7a 28  d1 9d 20 48 07 a9 63 76  |[.E.K.z(.. H..cv|
00000080  ac 2d 2b 3e bf af 5f 07  18 01 82 d7 b2 f3 ff f3  |.-+>.._.........|
00000090  ff fc 7f ff 7f 00 00 00  00 00 00 00 4b 18 58 02  |............K.X.|
```

[^1]: http://www.libpng.org/pub/png/book/chapter08.html#png.ch08.div.2