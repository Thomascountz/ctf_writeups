---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: [zsteg]
url: https://play.picoctf.org/practice/challenge/74
points: 150
captured: 2022-11-10
flag: picoCTF{h1d1ng_1n_th3_b1t5}
summary: Run `zsteg` on the image to reveal the flag.
---

> There's something in the building. Can you retrieve the flag?

> Given a challenge file, if we suspect steganography, we must do at least a little guessing to check if it's present. [Stegsolve (JAR download link)](http://www.caesum.com/handbook/Stegsolve.jar) is often used to apply various steganography techniques to image files in an attempt to detect and extract hidden data. You may also try [zsteg](https://github.com/zed-0xff/zsteg).[^1]

If we run `zsteg`, we get the following output.

```shell
$ zsteg buildings.png 
b1,r,lsb,xy         .. text: "^5>R5YZrG"
b1,rgb,lsb,xy       .. text: "picoCTF{h1d1ng_1n_th3_b1t5}"
b1,abgr,msb,xy      .. file: PGP Secret Sub-key -
b2,b,lsb,xy         .. text: "XuH}p#8Iy="
b3,abgr,msb,xy      .. text: "t@Wp-_tH_v\r"
b4,r,lsb,xy         .. text: "fdD\"\"\"\" "
b4,r,msb,xy         .. text: "%Q#gpSv0c05"
b4,g,lsb,xy         .. text: "fDfffDD\"\""
b4,g,msb,xy         .. text: "f\"fff\"\"DD"
b4,b,lsb,xy         .. text: "\"$BDDDDf"
b4,b,msb,xy         .. text: "wwBDDDfUU53w"
b4,rgb,msb,xy       .. text: "dUcv%F#A`"
b4,bgr,msb,xy       .. text: " V\"c7Ga4"
b4,abgr,msb,xy      .. text: "gOC_$_@o"
```

In this case, the output `b1,abgr,msb,xy`

[^1]: https://trailofbits.github.io/ctf/forensics/