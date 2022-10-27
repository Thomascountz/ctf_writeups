---
ctf: picoctf
competition: false
categories: forensics
tools: wireshark, dcode-cypher-identifier
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

>  As mentioned TFTP is designed to be implemented on top of the Datagram protocol (UDP).  Since Datagram is implemented on the Internet protocol, packets will have an Internet header, a Datagram header, and a TFTP header.  Additionally, the packets may have a header (LNI, ARPA header, etc.)  to allow them through the local transport medium. ...the order of the contents of a packet will be: local medium header, if used, Internet header, Datagram header, TFTP header, followed by the remainder of the TFTP packet. [^1]

We're interested in just the "...remainder of the TFTP packet", which here shows up as a string of ASCII characters in bytes `47-159`.

> **Note**
> We can also see TFTP opcode (`03` for DATA) and block number (`01`) in the TFTP in bytes `42-46`

Let's see what those ASCII characters are using dcode.fr's cypher identifier.

We get a hit for our friend, ROT13!

```
TFTPDOESNTENCRYPTOURTRAFFICSOWEMUSTDISGUISEOURFLAGTRANSFER.FIGUREOUTAWAYTOHIDETHEFLAGANDIWILLCHECKBACKFORTHEPLAN.
```

So `10.10.10.11` is telling `10.10.10.11` that they'll need to disguise the transfer of the flag somehow because TFTP isn't encrypted. They said they'll "...check back for the plan," so I wonder if we can find some other streams in the PcapNg file that show requests for a `plan` file.

| No. | Time         | Source      | Destination | Protocol | Length | Info                                                      |
| --- | ------------ | ----------- | ----------- | -------- | ------ | --------------------------------------------------------- |
| 9   | 3.499792153  | 10.10.10.11 | 10.10.10.12 | TFTP     | 60     | Read Request, File: plan, Transfer type: octet            |
| 10  | 0.000337875  | 10.10.10.12 | 10.10.10.11 | TFTP     | 61     | Error Code, Code: File not found, Message: File not found |
| 11  | 28.328590469 | 10.10.10.11 | 10.10.10.12 | TFTP     | 60     | Read Request, File: plan, Transfer type: octet            |
| 12  | 0.000350480  | 10.10.10.12 | 10.10.10.11 | TFTP     | 61     | Error Code, Code: File not found, Message: File not found |

Sure enough, we see read file requests followed by errors of opcode `01` - File not found.

> An ERROR packet can be the acknowledgment of any other type of packet. The error code is an integer indicating the nature of the error. [^1]

Then, we see another read request for `plan`, only this time, it's followed by an `Data Packet` response, an `Acknowledgement`, a request to read `program.deb`, and then `2865` blocks of data packets and acknowledgements...

| No.    | Time         | Source      | Destination | Protocol | Length | Info                                                  |
| ------ | ------------ | ----------- | ----------- | -------- | ------ | ----------------------------------------------------- |
| 19     | 11.726225781 | 10.10.10.11 | 10.10.10.12 | TFTP     | 60     | Read Request, File: plan, Transfer type: octet        |
| 20     | 0.000490193  | 10.10.10.12 | 10.10.10.11 | TFTP     | 105    | Data Packet, Block: 1 (last)                          |
| 21     | 0.000332354  | 10.10.10.11 | 10.10.10.12 | TFTP     | 60     | Acknowledgement, Block: 1                             |
| 22     | 5.042819037  | 10.10.10.11 | 10.10.10.12 | TFTP     | 62     | Read Request, File: program.deb, Transfer type: octet |
| 23     | 0.000564406  | 10.10.10.12 | 10.10.10.11 | TFTP     | 558    | Data Packet, Block: 1                                 |
| 24     | 0.000273528  | 10.10.10.11 | 10.10.10.12 | TFTP     | 60     | Acknowledgement, Block: 1                             |
| n      | 0.000122046  | 10.10.10.12 | 10.10.10.11 | TFTP     | 558    | Data Packet, Block: n                                 |
| n      | 0.000237184  | 10.10.10.11 | 10.10.10.12 | TFTP     | 60     | Acknowledgement, Block: n                             |
| 152412 | 0.000115029  | 10.10.10.12 | 10.10.10.11 | TFTP     | 558    | Data Packet, Block: 2865                              |
| 152413 | 0.008310560  | 10.10.10.11 | 10.10.10.12 | TFTP     | 60     | Acknowledgement, Block: 2865                          | 

Let's try to see what the `plan` was, i.e., what's in the data packet response?

```
0000   00 0c 29 79 6b b3 00 0c 29 66 7b ee 08 00 45 00   ..)yk...)f{...E.
0010   00 5b d4 17 40 00 40 11 3e 50 0a 0a 0a 0c 0a 0a   .[..@.@.>P......
0020   0a 0b a0 cd ac 7d 00 47 28 83 00 03 00 01 56 48   .....}.G(.....VH
0030   46 52 51 47 55 52 43 45 42 54 45 4e 5a 4e 41 51   FRQGURCEBTENZNAQ
0040   55 56 51 56 47 4a 56 47 55 2d 51 48 52 51 56 59   UVQVGJVGU-QHRQVY
0050   56 54 52 41 50 52 2e 50 55 52 50 58 42 48 47 47   VTRAPR.PURPXBHGG
0060   55 52 43 55 42 47 42 46 0a                        URCUBGBF.
```

This looks like more ROT13.

```
IUSEDTHEPROGRAMANDHIDITWITH-DUEDILIGENCE.CHECKOUTTHEPHOTOS
```



[^1]: https://www.rfc-editor.org/rfc/rfc1350