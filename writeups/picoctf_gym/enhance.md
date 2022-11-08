---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: [vim, grep]
url: https://play.picoctf.org/practice/challenge/265
points: 100
captured: 2022-11-08
flag: picoCTF{3nh4n3d_24374675}
summary: Open the SVG in a text editor to reveal the flag as a text element.
---

> Download this image file and find the flag.

SVGs are defined by a series of instructions in XML. When we open the image, we can see

Open up the SVG file in a text editor to reveal the flag as a `text` SVG element.

```shell
grep "tspan*" drawing.flag.svg 
       id="text3723"><tspan
         id="tspan3748">p </tspan><tspan
         id="tspan3754">i </tspan><tspan
         id="tspan3756">c </tspan><tspan
         id="tspan3758">o </tspan><tspan
         id="tspan3760">C </tspan><tspan
         id="tspan3762">T </tspan><tspan
         id="tspan3764">F { 3 n h 4 n </tspan><tspan
         id="tspan3752">c 3 d _ 2 4 3 7 4 6 7 5 }</tspan></text>
```

The flag is `picoCTF{3nh4nc3d_24374675}`