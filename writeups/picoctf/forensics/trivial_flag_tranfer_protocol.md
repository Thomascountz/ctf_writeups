---
ctf: picoctf
competition: false
categories: forensics
tools: wireshark, dcode-cypher-identifier, dpkg-deb, steghide
url: https://play.picoctf.org/practice/challenge/103
captured: 2022-10-27
flag: picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}
summary: Used Wireshark to inspect and download six files trasferred via TFTP. The first two files described instructions and a plan encoded via ROT13. Next, the steghide Debian package and three image files were transferred. Using steghide and the passphrase from the plan file, one of the three image files contained the flag.
---

# Trivial Flag Transfer Protocol

> Figure out how they moved the flag

We're given a large large (`50MB`) PcapNg file that we can open immediately in Wireshark.

There are a lot of TFTP (_Trivial_ File Transfer Protocol) streams.

> Overview of the Protocol
> Any transfer begins with a request to read or write a file, which also serves to request a connection.  If the server grants the request, the connection is opened and the file is sent in fixed length blocks of 512 bytes.  Each data packet contains one block of data, and must be acknowledged by an acknowledgment packet before the next packet can be sent.  A data packet of less than 512 bytes signals termination of a transfer... Notice that both machines involved in a transfer are considered senders and receivers.  One sends data and receives acknowledgments, the other sends acknowledgments and receives data. [^1]

Let's look at the first series of streams.

| No. | Time        | Source      | Destination | Protocol | Length | Info                                                        |
| --- | ----------- | ----------- | ----------- | -------- | ------ | ----------------------------------------------------------- |
| 1   | 0.000000000 | 10.10.10.11 | 10.10.10.12 | TFTP     | 67     | Write Request, File: instructions.txt, Transfer type: octet |
| 2   | 0.000451771 | 10.10.10.12 | 10.10.10.11 | TFTP     | 46     | Acknowledgement, Block: 0                                   |
| 3   | 0.000312738 | 10.10.10.11 | 10.10.10.12 | TFTP     | 159    | Data Packet, Block: 1 (last)                                |
| 4   | 0.000111406 | 10.10.10.12 | 10.10.10.11 | TFTP     | 46     | Acknowledgement, Block: 1                                   |

Here, we see the request to write a file called `instructions.txt`, which "...also serves to request a connection" to `10.10.10.12` from `10.10.10.11` (1).  That packet is acknowledge, and the connection is established (2). Then a one block data packet is sent of size `159b`, which is less than `512b`, so it also signals the termination of the transfer (3). The stream ends with a final acknowledgement packet (4).

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

Let's see what those ASCII characters are using [dcode.fr](https://www.dcode.fr/en)'s cypher identifier.

We get a hit for our friend, ROT13!

```
TFTPDOESNTENCRYPTOURTRAFFICSOWEMUSTDISGUISEOURFLAGTRANSFER.FIGUREOUTAWAYTOHIDETHEFLAGANDIWILLCHECKBACKFORTHEPLAN
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

Then, we see another read request for `plan`, only this time, it's followed by a `Data Packet` response, an `Acknowledgement`, a request to read `program.deb`, and then hundreds of blocks of data packets and acknowledgements...

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

Let's try to see what the `plan` was, i.e. what's in the data packet response?

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

Let's take a look at the rest of the requests.

| No.    | Time        | Source      | Destination | Protocol | Length | Info                                                   |
| ------ | ----------- | ----------- | ----------- | -------- | ------ | ------------------------------------------------------ |
| 22     | 5.042819037 | 10.10.10.11 | 10.10.10.12 | TFTP     | 62     | Read Request, File: program.deb, Transfer type: octet  |
| 567    | 3.711624110 | 10.10.10.11 | 10.10.10.12 | TFTP     | 63     | Read Request, File: picture1.bmp, Transfer type: octet |
| 3790   | 3.727568743 | 10.10.10.11 | 10.10.10.12 | TFTP     | 63     | Read Request, File: picture2.bmp, Transfer type: octet |
| 146683 | 2.818313205 | 10.10.10.11 | 10.10.10.12 | TFTP     | 63     | Read Request, File: picture3.bmp, Transfer type: octet |

We see the request from earlier for the `program.deb` file, and then we see three requests for `bmp` image files. All of them are transferred without error.

With Wireshark, we can extract/export/download objects transferred via different protocols. For us, we can extract TFTP objects to download the six files that we transferred.

```
File -> Export Objects -> TFTP...
```

![](./attachments/Screen%20Shot%202022-10-27%20at%2011.13.43%20PM.png)

The `plan` said "IUSEDTHEPROGRAM..." so let's find out what `program.deb` is.

> A DEB file is a software package used by the Debian [Linux](https://techterms.com/definition/linux) distribution and its variants, such as Ubuntu. DEB files are used primarily to install or update [Unix](https://techterms.com/definition/unix) applications. [^2]

We can use the tool `dpkg-deb` to do things like inspect the file contents, extract their contents, and list metadata about the package.

```shell
$ dpkg --info program.deb 
 new Debian package, version 2.0.
 size 138310 bytes: control archive=1250 bytes.
     826 bytes,    18 lines      control              
    1184 bytes,    17 lines      md5sums              
 Package: steghide
 Source: steghide (0.5.1-9.1)
 Version: 0.5.1-9.1+b1
 Architecture: amd64
 Maintainer: Ola Lundqvist <opal@debian.org>
 Installed-Size: 426
 Depends: libc6 (>= 2.2.5), libgcc1 (>= 1:4.1.1), libjpeg62-turbo (>= 1:1.3.1), libmcrypt4, libmhash2, libstdc++6 (>= 4.9), zlib1g (>= 1:1.1.4)
 Section: misc
 Priority: optional
 Description: A steganography hiding tool
  Steghide is steganography program which hides bits of a data file
  in some of the least significant bits of another file in such a way
  that the existence of the data file is not visible and cannot be proven.
  .
  Steghide is designed to be portable and configurable and features hiding
  data in bmp, wav and au files, blowfish encryption, MD5 hashing of
  passphrases to blowfish keys, and pseudo-random distribution of hidden bits
  in the container data.
```

Ok! We already have `steghide` installed (and even if we didn't, I wouldn't trust installing it using this `.deb` package), so let's see what happens when we run it on `.bmp` files.

```shell
$ steghide extract -sf picture1.bmp 
Enter passphrase:
```

Aha! It's asking for a passphrase. "...ANDHIDITWITH-DUEDILIGENCE..."

Let's try it.

```shell
$ steghide extract -sf picture1.bmp --passphrase DUEDILIGENCE
steghide: could not extract any data with that passphrase!

$ steghide extract -sf picture2.bmp --passphrase DUEDILIGENCE
steghide: premature end of file "picture2.bmp" while reading bmp data.

$ steghide extract -sf picture3.bmp --passphrase DUEDILIGENCE
wrote extracted data to "flag.txt".
```

Let's see what's in `flag.txt`.

```shell
$ cat flag.txt 
picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}
```

[^1]: https://www.rfc-editor.org/rfc/rfc1350
[^2]: https://fileinfo.com/extension/deb 