---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: [wireshark]
url: https://play.picoctf.org/practice/challenge/286
points: 100
captured: 2022-11-09
flag: picoCTF{p4ck37_5h4rk_b9d53765}
summary: Manually scan through the nine packets in wireshark to find the flag within the data of a TCP packet.
---

> Download the packet capture file and use packet analysis software to find the flag.

The `.pcap` file only has nine packets in it.

![packets_primer](./attachments/packets_primer.png)

We can manually scan them and find the flag within the data of a TCP packet.

![packets_primer_flag](./attachments/packets_primer_flag.png)