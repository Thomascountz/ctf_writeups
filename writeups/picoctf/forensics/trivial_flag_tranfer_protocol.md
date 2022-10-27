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
> Any transfer begins with a request to read or write a file, which also serves to request a connection.  If the server grants the request, the connection is opened and the file is sent in fixed length blocks of 512 bytes.  Each data packet contains one block of data, and must be acknowledged by an acknowledgment packet before the next packet can be sent.  A data packet of less than 512 bytes signals termination of a transfer... Notice that both machines involved in a transfer are considered senders and receivers.  One sends data and receives acknowledgments, the other sends acknowledgments and receives data. [^1]

Let's look at the first stream.

| No. | Time        | Source      | Destination | Protocol | Length | Info                                                        |
| --- | ----------- | ----------- | ----------- | -------- | ------ | ----------------------------------------------------------- |
| 1   | 0.000000000 | 10.10.10.11 | 10.10.10.12 | TFTP     | 67     | Write Request, File: instructions.txt, Transfer type: octet |
| 2   | 0.000451771 | 10.10.10.12 | 10.10.10.11 | TFTP     | 46     | Acknowledgement, Block: 0                                   |
| 3   | 0.000312738 | 10.10.10.11 | 10.10.10.12 | TFTP     | 159    | Data Packet, Block: 1 (last)                                |
| 4   | 0.000111406 | 10.10.10.12 | 10.10.10.11 | TFTP     | 46     | Acknowledgement, Block: 1                                   |

Here, we see (1) the request to write a file called `instructions.txt`, which "...also serves to request a connection" to `10.10.10.12` from `10.10.10.11`.  (2) That packet is acknowledge, and the connection is established. (3) Then a one block data packet is sent of size `159b`, which is less than `512b`, so it also signals the termination of the transfer. (4) The stream ends with a final acknowledgement packet.

Let's take a look at the data that was transferred in the data packet.

```
0000   00 0c 29 66 7b ee 00 0c 29 79 6b b3 08 00 45 00   ..)f{...)yk...E.
0010   00 91 79 b8 40 00 40 11 98 79 0a 0a 0a 0b 0a 0a   ..y.@.@..y......
0020   0a 0c 87 e6 cc 83 00 7d 78 be 00 03 00 01 47 53   .......}x.....GS
0030   47 43 51 42 52 46 41 47 52 41 50 45 4c 43 47 42   GCQBRFAGRAPELCGB
0040   48 45 47 45 4e 53 53 56 50 46 42 4a 52 5a 48 46   HEGENSSVPFBJRZHF
0050   47 51 56 46 54 48 56 46 52 42 48 45 53 59 4e 54   GQVFTHVFRBHESYNT
0060   47 45 4e 41 46 53 52 45 2e 53 56 54 48 45 52 42   GENAFSRE.SVTHERB
0070   48 47 4e 4a 4e 4c 47 42 55 56 51 52 47 55 52 53   HGNJNLGBUVQRGURS
0080   59 4e 54 4e 41 51 56 4a 56 59 59 50 55 52 50 58   YNTNAQVJVYYPURPX
0090   4f 4e 50 58 53 42 45 47 55 52 43 59 4e 41 0a      ONPXSBEGURCYNA.
```

[^1]: https://www.rfc-editor.org/rfc/rfc1350#section-2). Overview of the Protocol