---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: [wireshark]
url: https://play.picoctf.org/practice/challenge/30
points: 150
captured: 2022-11-09
flag: picoCTF{StaT31355_636f6e6e}
summary: Use Wireshark to find the UDP stream where the flag was sent one byte at a time.
---

Opening up the `.pcap` file in Wireshark, the first thing we can do is have a look at the 