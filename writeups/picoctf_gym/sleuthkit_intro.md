---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: [file, gzip, mmls, nc]
url: https://play.picoctf.org/practice/challenge/301
points: 100
captured: 2022-11-09
flag: picoCTF{mm15_f7w!}
summary: Run `mmls` on the disk image decompressed using `gzip` to get the size of the linux partition. Pass that value into the checker service to get the flag.
---

 > Download the disk image and use `mmls` on it to find the size of the Linux partition. Connect to the remote checker service to check your answer and get the flag. 

```shell
$ file disk.img.gz 
disk.img.gz: gzip compressed data, was "disk.img", last modified: Tue Sep 21 19:34:53 2021, from Unix, original size modulo 2^32 104857600

$ gzip -d disk.img.gz 

$ mmls disk.img 
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000204799   0000202752   Linux (0x83)

$ nc saturn.picoctf.net 52279
What is the size of the Linux partition in the given disk image?
Length in sectors: 202752
202752
Great work!
picoCTF{mm15_f7w!}
```