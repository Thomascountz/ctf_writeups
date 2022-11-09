---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: [exiftool]
url: https://play.picoctf.org/practice/challenge/19
points: 150
captured: 2022-11-09
flag: picoCTF{s0_m3ta_fec06741}
summary: Use `exiftool` to find the flag in the `Artist` metadata.
---

```shell
$exiftool pico_img.png 
ExifTool Version Number         : 12.16
File Name                       : pico_img.png
Directory                       : .
...
Warning                         : [minor] Text chunk(s) found after PNG IDAT (may be ignored by some readers)
Artist                          : picoCTF{s0_m3ta_fec06741}
Image Size                      : 600x600
Megapixels                      : 0.360
```