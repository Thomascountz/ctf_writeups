---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: [strings]
url: https://play.picoctf.org/practice/challenge/44
points: 50
captured: 2022-10-25
flag: picoCTF{more_than_m33ts_the_3y3657BaB2C} 
summary: Used `strings` to the find human-readable plaintext flag in a jpg file.
---

# Glory in the Garden

> This garden contains more than it seems.

Trying `strings` revealed the flag.

```shell
$ strings garden.jpg | tail -n 10
h--@3
cZi-
M(.I
]hWP&
jc#k
=7g&
mjx/
s\]|."Ue
\qZf
Here is a flag "picoCTF{more_than_m33ts_the_3y3657BaB2C}"
```