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

> Overview of the Protocol
> Any transfer begins with a request to read or write a file, which also serves to request a connection.  If the server grants the request, the connection is opened and the file is sent in fixed length blocks of 512 bytes.  Each data packet contains one block of data, and must be acknowledged by an acknowledgment packet before the next packet can be sent.  A data packet of less than 512 bytes signals termination of a transfer.  ...Notice that both machines involved in a transfer are considered senders and receivers.  One sends data and receives acknowledgments, the other sends acknowledgments and receives data. [^1]

Let's look at the first 

[^1]: https://www.rfc-editor.org/rfc/rfc1350#section-2). Overview of the Protocol