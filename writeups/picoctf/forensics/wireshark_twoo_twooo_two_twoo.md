---
ctf: picoctf
competition: false
categories: [forensics]
tools: [wireshark, dcode-cipher-identifier]
url: https://play.picoctf.org/practice/challenge/110
captured: 
flag: 
summary:
---

> Can you find the flag? shark2.pcapng.

Let's open the file in Wireshark to see what we're dealing with. We'll start with our HTTP 200 filter.

```
http.response.code == 200
```

There's a lot of encrypted traffic that looks similar.

| No. | Time     | Source          | Destination    | Protocol | Length | Info                                                         |
| --- | -------- | -------------- | -------------- | -------- | ------ | ------------------------------------------------------------ |
| 40  | 0.000000 | 192.168.38.103 | 192.168.38.104 | HTTP     | 553    | HTTP/1.1 200   (application/http-kerberos-session-encrypted) |

But there's also a series of plaintext traffic to and from the same destination and source.

| No. | Time     | Source       | Destination    | Protocol | Length | Info                         |
| --- | -------- | ----------- | -------------- | -------- | ------ | ---------------------------- |
| 57  | 0.000116 | 18.217.1.57 | 192.168.38.104 | HTTP     | 289    | HTTP/1.0 200 OK  (text/html) |

Let's look at the first occurance.

```
0000   02 3b c6 1a ae f5 02 fb 68 4c e9 41 08 00 45 00   .;......hL.A..E.
0010   01 13 56 94 40 00 24 06 04 2f 12 d9 01 39 c0 a8   ..V.@.$../...9..
0020   26 68 00 50 f8 94 e1 a7 3f e0 97 8b 96 e6 50 19   &h.P....?.....P.
0030   01 e7 a9 56 00 00 43 6f 6e 74 65 6e 74 2d 54 79   ...V..Content-Ty
0040   70 65 3a 20 74 65 78 74 2f 68 74 6d 6c 3b 20 63   pe: text/html; c
0050   68 61 72 73 65 74 3d 75 74 66 2d 38 0d 0a 43 6f   harset=utf-8..Co
0060   6e 74 65 6e 74 2d 4c 65 6e 67 74 68 3a 20 39 39   ntent-Length: 99
0070   0d 0a 53 65 72 76 65 72 3a 20 57 65 72 6b 7a 65   ..Server: Werkze
0080   75 67 2f 31 2e 30 2e 31 20 50 79 74 68 6f 6e 2f   ug/1.0.1 Python/
0090   33 2e 36 2e 39 0d 0a 44 61 74 65 3a 20 4d 6f 6e   3.6.9..Date: Mon
00a0   2c 20 31 30 20 41 75 67 20 32 30 32 30 20 30 31   , 10 Aug 2020 01
00b0   3a 33 39 3a 31 36 20 47 4d 54 0d 0a 0d 0a 54 68   :39:16 GMT....Th
00c0   65 20 6f 66 66 69 63 69 61 6c 20 52 65 64 27 73   e official Red's
00d0   20 53 68 72 69 6d 70 20 61 6e 64 20 48 65 72 72    Shrimp and Herr
00e0   69 6e 67 20 77 65 62 73 69 74 65 20 69 73 20 73   ing website is s
00f0   74 69 6c 6c 20 75 6e 64 65 72 20 63 6f 6e 73 74   till under const
0100   72 75 63 74 69 6f 6e 2e 20 50 6c 65 61 73 65 20   ruction. Please 
0110   63 68 65 63 6b 20 62 61 63 6b 20 6c 61 74 65 72   check back later
0120   21                                                !
```

> The official Red's Shrimp and Herring website is still under construction. Please check back later!

"Check back later..." Let's look at the next request.

|No.|Time    |Source      |Destination   |Protocol|Length|Info                        |
|---|--------|-----------|--------------|--------|------|----------------------------|
|744|0.000037|18.217.1.57|192.168.38.104|HTTP    |263   |HTTP/1.0 200 OK  (text/html)|

```
0000   02 3b c6 1a ae f5 02 fb 68 4c e9 41 08 00 45 00   .;......hL.A..E.
0010   00 f9 96 20 40 00 24 06 c4 bc 12 d9 01 39 c0 a8   ... @.$......9..
0020   26 68 00 50 f8 95 a3 7f 71 ef bb 0b 06 d5 50 19   &h.P....q.....P.
0030   01 e7 bd 67 00 00 43 6f 6e 74 65 6e 74 2d 54 79   ...g..Content-Ty
0040   70 65 3a 20 74 65 78 74 2f 68 74 6d 6c 3b 20 63   pe: text/html; c
0050   68 61 72 73 65 74 3d 75 74 66 2d 38 0d 0a 43 6f   harset=utf-8..Co
0060   6e 74 65 6e 74 2d 4c 65 6e 67 74 68 3a 20 37 33   ntent-Length: 73
0070   0d 0a 53 65 72 76 65 72 3a 20 57 65 72 6b 7a 65   ..Server: Werkze
0080   75 67 2f 31 2e 30 2e 31 20 50 79 74 68 6f 6e 2f   ug/1.0.1 Python/
0090   33 2e 36 2e 39 0d 0a 44 61 74 65 3a 20 4d 6f 6e   3.6.9..Date: Mon
00a0   2c 20 31 30 20 41 75 67 20 32 30 32 30 20 30 31   , 10 Aug 2020 01
00b0   3a 33 39 3a 31 39 20 47 4d 54 0d 0a 0d 0a 70 69   :39:19 GMT....pi
00c0   63 6f 43 54 46 7b 62 66 65 34 38 65 38 35 30 30   coCTF{bfe48e8500
00d0   63 34 35 34 64 36 34 37 63 35 35 61 34 34 37 31   c454d647c55a4471
00e0   39 38 35 65 37 37 36 61 30 37 62 32 36 63 62 61   985e776a07b26cba
00f0   36 34 35 32 36 37 31 33 66 34 33 37 35 38 35 39   64526713f4375859
0100   39 61 61 39 38 62 7d                              9aa98b}
```

There's a flag there!

```
picoCTF{bfe48e8500c454d647c55a4471985e776a07b26cba64526713f43758599aa98b}
```

Hmmm... That flag didn't work. Maybe it's encoded. Let's try [https://www.dcode.fr/](https://www.dcode.fr/cipher-identifier)