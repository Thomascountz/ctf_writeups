---
ctf: picoctf
competition: false
categories: forensics
tools: wireshark
url: https://play.picoctf.org/practice/challenge/103
captured: 
flag: 
summary:
---

# Trivial Flag Transfer Protocol

> Figure out how they moved the flag

The file is a large (`50MB`) PcapNg, which we've seen in [wireshark_doo_dooo_do_doo](wireshark_doo_dooo_do_doo.md).

There are a lot of TFTP (_Trivial_ File Transfer Protocol) streams.