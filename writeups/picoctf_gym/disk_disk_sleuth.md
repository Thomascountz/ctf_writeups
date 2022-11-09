---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: [gzip, srch_strings, grep]
url: https://play.picoctf.org/practice/challenge/113
points: 110
captured: 2022-11-09
flag: picoCTF{f0r3ns1c4t0r_n30phyt3_a011c142}
summary: Decompress the disk image and use `srch_strings` and `grep` to find the flag.
---


```shell
$ gzip -d dds1-alpine.flag.img.gz 

$ srch_strings dds1-alpine.flag.img | grep "pico"
ffffffff81399ccf t pirq_pico_get
ffffffff81399cee t pirq_pico_set
ffffffff820adb46 t pico_router_probe
  SAY picoCTF{f0r3ns1c4t0r_n30phyt3_a011c142}
```