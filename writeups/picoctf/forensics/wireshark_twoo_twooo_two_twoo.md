---
ctf: picoctf
competition: false
categories: [forensics]
tools: [wireshark, dcode-cipher-identifier, cyberchef]
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

Let's look at the first occurrence.

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

"Check back later..." Let's look at the next request. This one is to a  `/flag` endpoint!

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

Hmmm... That flag didn't work. Maybe it's encoded. Let's try [https://www.dcode.fr/](https://www.dcode.fr/cipher-identifier)].

It looks like it's hexadecimal (this can be caught by eye by noticing there are only number `0`-`9` and letter `A`-`F`).

Let's convert it.

```
¿ä...ÄTÖGÅZDq.^wj.²lºdRg.ô7XY.©.
```

Hmmm... That doesn't mean much to me and doesn't work as a flag either.

Let's take a look at the next request.

| No.  | Time     | Source      | Destination    | Protocol | Length | Info                         |
| ---- | -------- | ----------- | -------------- | -------- | ------ | ---------------------------- |
| 1550 | 0.000129 | 18.217.1.57 | 192.168.38.104 | HTTP     | 263    | HTTP/1.0 200 OK  (text/html) |

```
0000   02 3b c6 1a ae f5 02 fb 68 4c e9 41 08 00 45 00   .;......hL.A..E.
0010   00 f9 45 6b 40 00 24 06 15 72 12 d9 01 39 c0 a8   ..Ek@.$..r...9..
0020   26 68 00 50 f8 97 e7 34 dc 6e 38 18 04 50 50 19   &h.P...4.n8..PP.
0030   01 e7 b6 96 00 00 43 6f 6e 74 65 6e 74 2d 54 79   ......Content-Ty
0040   70 65 3a 20 74 65 78 74 2f 68 74 6d 6c 3b 20 63   pe: text/html; c
0050   68 61 72 73 65 74 3d 75 74 66 2d 38 0d 0a 43 6f   harset=utf-8..Co
0060   6e 74 65 6e 74 2d 4c 65 6e 67 74 68 3a 20 37 33   ntent-Length: 73
0070   0d 0a 53 65 72 76 65 72 3a 20 57 65 72 6b 7a 65   ..Server: Werkze
0080   75 67 2f 31 2e 30 2e 31 20 50 79 74 68 6f 6e 2f   ug/1.0.1 Python/
0090   33 2e 36 2e 39 0d 0a 44 61 74 65 3a 20 4d 6f 6e   3.6.9..Date: Mon
00a0   2c 20 31 30 20 41 75 67 20 32 30 32 30 20 30 31   , 10 Aug 2020 01
00b0   3a 33 39 3a 32 33 20 47 4d 54 0d 0a 0d 0a 70 69   :39:23 GMT....pi
00c0   63 6f 43 54 46 7b 62 64 61 36 39 62 64 66 38 66   coCTF{bda69bdf8f
00d0   35 37 30 61 39 61 61 61 62 30 65 34 31 30 38 61   570a9aaab0e4108a
00e0   30 66 61 35 66 36 34 63 62 32 36 62 61 37 64 32   0fa5f64cb26ba7d2
00f0   32 36 39 62 62 36 33 66 36 38 61 66 35 64 39 38   269bb63f68af5d98
0100   62 39 38 32 34 35 7d                              b98245}
```

Another flag!

```
picoCTF{bda69bdf8f570a9aaab0e4108a0fa5f64cb26ba7d2269bb63f68af5d98b98245}
```

Another false flag and nonsense once decoded from hexadecimal.

```
½¦ßW
ª°ä¥öL²k§Ò&¶?h¯]¹E
```

Sure enough, they all seem to contain false hexadecimal flags... 

```
picoCTF{bfe48e8500c454d647c55a4471985e776a07b26cba64526713f43758599aa98b}
picoCTF{bda69bdf8f570a9aaab0e4108a0fa5f64cb26ba7d2269bb63f68af5d98b98245}
picoCTF{0a024b7d39603756feafa2bbaa1603b14a99eae5dcd59f1d957f511d822c8c06}
picoCTF{97211eec9228bb247d762527bace8b3e4ec2110c8834af12aefd3c552cdc21b2}
picoCTF{29679910c47d8afc737a1c21d7bf758cd3d81001bdbeec8c6f81a6ad88fdc279}
picoCTF{996979e9540be0fe9320e80eb6336047f8140a80830700907b99741310acf08f}
picoCTF{8b272a18c1005c95a420d4a0df426cb8441d29eb96210493a96fa25ac5e657aa}
picoCTF{e1d0a752dc71121200f4bcb1b8cc2e03e84488df229b82196afbe0045ef025c4}
picoCTF{0ba511844a2ab38fe0709bcdb2b8bdfeb37a0b466dc902e92062db4c2b3f455c}
picoCTF{dadda48e855421e14597ffc727943b57efd8c9a15d10bfd491f0390659162fb1}
picoCTF{f4dd87795395c74f3083f8caa4ec22d1531281554a6003d1c47c5f0370984ab6}
picoCTF{0f30a584680db9e70c7e1c6ca954c2f023b77f3fd2b05bd9aeee6e00dc4da5d7}
picoCTF{fe83bcb6cfd43d3b79392f6a4232685f6ed4e7a789c2ce559cf3c1ab6adbe34b}
picoCTF{715e4d0d167e862af8825f62d3f4ff8aef20443445a06b1c68572390a2825d29}
picoCTF{7654ee03f31576e8ed44799fc4fa5ee053d35050000502e878d1fb8022618923}
picoCTF{068606b5faca0491d97a2b46fdca7f6f81acbd909ce691077fe77e03a3c0939a}
picoCTF{64ab681ffed33c49b5e8ae0576e22857e9a10ae30cdbee415fb514b84aa58aea}
picoCTF{8ae3995e726f8f2c3724e2e0522f038aba6649facd378d8965c648233d79a252}
picoCTF{1c125d267b5811cd25cca2d517e022270aa60f3c8461f4097c685bcca637a6a9}
picoCTF{824c298d14e1fe369df991af72ab0725d2e7c7d05b9655486873ccc467f4bd6b}
picoCTF{e1d8dd1b73d5fd7704a16c924ddee69dc6bf9beef14cc3a10142704b81f0fa07}
picoCTF{82d260fe0670d551347b164c54183d996c52ebeebb1ccfcc2c2ebb91268dc944}
picoCTF{74876fc61ebc9c902f8983979cd4c21206c69a23f0dcc0817e150dd75e446838}
picoCTF{711d3893d90f100c15e10ef4842abeed3a830f8237c1257cd47389646da97810}
picoCTF{49c52d1f30973f90716bbcbe3633e11cf70b9a31ed785871ccb80473302a59db}
picoCTF{89d93dbb96a3857ac87ba0cea3c10a9e4c7b34d79b2edb463cef030d34297bd0}
picoCTF{5ceacdce54c13a3fddfcfb225a00247304fbb15f29f9c90434383f277567992d}
picoCTF{c22a40a43ed7034bd935805f59603a46d3a1f2d6b8e31281eb0721597b6c6d62}
picoCTF{6071bca5da06d4f975a52357cda0cd6f0614787c1c70b1b7e1af2c7fb272d281}
picoCTF{65a8b141f019506feea38a119988ad645bcab1a5fa8693efdf26e1fd3cb44b4c}
picoCTF{d7f5cb78a895d3805601522b95d599cb6d2689c6a856e3fbee6aac2fca0c20f3}
picoCTF{739bb0f0aa17331819a0e942d37bfee757c8d9cd089cdfe32509027b92485213}
picoCTF{7a891e2c4ad0da374bc15ad7ad0ee081077dd376f06152781f780c201691713d}
picoCTF{a97d3ee943221888bd1157429e4a00ed5e9905a610e64664f7e36c7f5e0a4ef9}
picoCTF{3cf1e22d489fcfb6bb312a34f46c8699989ed043406134331452d11ce73cd59e}
picoCTF{c38d2d74dc21bbb2e3a95b52e2354ee523379cfe4f8b348c9c5b5d7bd7cb871b}
picoCTF{e4dc886c39a53ff118bf29041067cde48dcebb89b3dae61a8aba6187d671999a}
picoCTF{9fbd0d18aa1abfd289ba977ae4354b821cc74591260889afba1b0b6e7763aa31}
picoCTF{3fc0801bcd36336a2c030c6e5f452f5795be1d562e00411365fb64c6a2f688ef}
picoCTF{4aa86643eb2ddb5709725344cd0e63e6c52e35c2e64a39f3a4a0ee7bbd5d3ade}
picoCTF{4af8df415d17e6df99a5efddebcb33a68c0c8bf26d481eed16b5f77675030d7f}
picoCTF{e4f52a0d2a924906ac102a32c52ab9128bf9cd6e5294518ad3ed6748f853b0ab}
picoCTF{cc104e74a9f50164ee5652d168ef38a21b7a2d5e3196062e669e3a2705f1a0d3}
picoCTF{2aac620b0bdd2e6946d62c5d232ca32ba1f5a9d8ec82c060778b54ffeb8fbd1f}
picoCTF{4e55be07159def207afc142954f5673a0651d5f32f5f4090fb774d960628e352}
picoCTF{b4cc138bb0f7f9da7e35085e349555aa6d00bdca3b021c1fe8663c0a422ce0d7}
picoCTF{983e5e2703a132a49479e438bfba15ee5d02345b03d410b8163b685973937da7}
picoCTF{d342a46e8179de9941720c5e0eeac0d0fae9d3014d2ddcf531a7865a997b00e5}
picoCTF{2133904cfe757bc6c68c3e5f3749b37d67d7fa6ffb2768410be593d3fe8c4bd4}
picoCTF{29b726b9a57d176e1487d159474ee7e6508b66c05c526a00c942a8cebb6bb496}
picoCTF{7302b0dca07cd890c75e38d78d7e74d7bbf2b932f555aaf5b6754f56e778e3fc}
picoCTF{22e018bb8282e9d7852ed4e65f70a26524dabef78cf41e1db45c070c94621c57}
picoCTF{40f366ccf0f6462f5b8b1dc4d7384a62aa95565afcaad96a937b8c1f1134099b}
picoCTF{db38cbc215cde0d9cd52cbca2390defdb54303e998019a5c4ddaf9861b54efcb}
picoCTF{090fa8ec995ab9fc9f97cbe9ea36cb81c4504a3ca02466ddd207cfe7f785cb5c}
picoCTF{947b91a983c93217304f8e5b112e93eaf619e6a9386ab93be93a9b67e53b2fda}
picoCTF{41b8a1a796bd8d202016f75bc5b38889e9ea06007e6b22fc856d380fb7573133}
picoCTF{a3ed2f602322f749f4cb016515e25b67749efd08ac2f2c53023596cbf0dcbd0f}
picoCTF{8e625859eb325d2a69934e4a44c93fcc132e813efb3fdaaa5143147678e9cbf9}
picoCTF{8d43c4889ee5b507d1785adfa2592f2fb3d7cf20ebf37ce46595edc46fba3f6d}
picoCTF{0020d021e9e38dbb5a5fa432175089d8b76e4a900618c95f8cae14fedaa45b63}
picoCTF{69e96b10f560a6a0656a6d950e73e41bcf4226c424bb5622839dda0c66755b14}
picoCTF{34c6ca47d858ab18aa2008f4ac31c31570c46186939e6b46458b19082122d4bd}
picoCTF{ebfcebe696b1fdbba2abb3b003165152456bd83b6ddfbf180ca366de0dec1b0c}
picoCTF{aa125aaeb4723f69dceaa90125a8099a6f3fe0259e068fd82dcbeb76131448bb}
picoCTF{80d65857d8d81a92769e8cd136376522d113c4298b331318ce7adcbf5e70104d}
picoCTF{00ae773ce4a4b3cf3287f072c13ec7139a74207de635de9d115087bc4f312bae}
picoCTF{9812bc4be04e6f9c803152313db3da53b3dfb799bdb05aac46fa0dd0045d2fc2}
picoCTF{7e808778b7250893922a17d53f10365b009a7624935850ac5c8140461e49d579}
picoCTF{33e80d6e9f56c1f7705c73566d347ccb32b4662171f224b6dfcb6c8fce4f1601}
picoCTF{5d921ffbe2709ba82d09603a095530aedae41ab96fd052140cbc64319b7ab0ac}
picoCTF{977b385d5dd6abde9cb89ee940b5cfb7179d73d989c6993346d278bff003c154}
picoCTF{ca7d3b029817de8f318d8fa521ad1b569f4e8a37358373193522cc7f5628ed49}
picoCTF{a820680ab6444b1daf5281192f337aefb4aa95a313c9f270804ef7826ecc298c}
picoCTF{998d01dadf1b44eb4ec7b7e8fa11f11bcd2d7d86f3f9e4966dde22d4a84ca113}
picoCTF{cb8fe3ec65f890e2f0570c98c4edd3fe4115bc059ac2afb39300c7b66f2302c4}
picoCTF{bc2af8cbe0ae0befdd28b14412295243354cd3c7cc74e88d8facb2fd5e6ef34d}
picoCTF{09082a0313e16fc36f8076ff86e54e83048a8568f5c2294fea5fb3bcd212e7f2}
picoCTF{64cf3ede3736a340fdf2954be5151ce53bec291c5e48cbccb44faa529946e249}
picoCTF{2386746aeb258914349dc81a85cb5de72e47930c7f11759b4ad9f864efa7b5aa}
picoCTF{173306d7b886423d9f79d3d0d05209807ae7b83c445931319830e4e0ad2d2f09}
picoCTF{6cb98e2295bbe1f15fd8b8b5908de360d386b98a0ce7e0407e001b453b05be22}
picoCTF{132e643c8fdadb54c366072cb33940411fcfd355209fc1ce9b2022ad1cd1b060}
picoCTF{044ffca72f0f191b0715ff1a9bff182c810cb2786370cbf8cdc1943c2e7aedf6}
picoCTF{b278104c2602442e3db401749c30527d80ba560f9a02c939cb4ff6ea189a140d}
picoCTF{7282e048d6d32383b65f3a03b1101219ac73f7f538446b78d1b2b334e0985447}
picoCTF{98406c4acbf0f57b3ccbc923aab5a603d70f86d507f422d9bd8656398f53433e}
picoCTF{3fe0b2788f30d9cb9f77d3b2752f13c554fe7f0e7a2883e57c8a44b34f35675c}
picoCTF{c50d259a4e172fcb2eddbabeebd272473e4882b76c9efcd12c03ac04429d884a}
```

This is where I got a bit stuck. These flags were red herrings (as evidenced by the "Red's Shrimp and Herring" play on words). 

There are other streams logged in the PcapNg file, but what about requests fro