---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: [grep]
url: https://play.picoctf.org/practice/challenge/279
points: 100
captured: 2022-11-09
flag: picoCTF{gr3p_15_@w3s0m3_58f5c024}
summary: Use `grep` to find the flag in a large 107KB text file.
---

> Attackers have hidden information in a very large mass of data in the past, maybe they are still doing it. Download the data here.

Once we download the file, we can see that it's just a text file.

```shell
$ file anthem.flag.txt 
anthem.flag.txt: UTF-8 Unicode text
```

We can see how large it is using `ls -lh` to discover it's `107KB`.

```shell
$ ls -lh anthem.flag.txt 
-rw-r--r-- 1 simon simon 107K Nov  9 16:57 anthem.flag.txt
```

If we grep for `pico`, we find the flag.

```shell
grep -i pico anthem.flag.txt 
      we think that the men of picoCTF{gr3p_15_@w3s0m3_58f5c024}
```