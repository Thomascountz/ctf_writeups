---
ctf: picoctf
competition: false
categories: [forensics]
tools: [exiftool, strings, png-file-chunck-inspector, cyberchef, unzip]
url: https://play.picoctf.org/practice/challenge/129
captured: 2022-10-24
flag: picoCTF{336cf6d51c9d9774fd37196c1d7320ff}
summary: Used exiftool to reveal an archive file carved into a png after the IEND chunk that then could be unzipped to reveal the plaintext flag file.
---

# Matryoshka Doll

> Matryoshka dolls are a set of wooden dolls of decreasing size placed one inside another. What's the final one? Image: this

We're asked to download an image `dolls.jpg`.

If we look at the MIME type of the file, it's `image/png`, but the extension is `.jpg`.

```shell
$ exiftool dolls.jpg | grep MIME
MIME Type                       : image/png
```

If we look at the rest of the `exiftool` output, we see a `Warning`.

```shell 
$ exiftool dolls.jpg | grep Warning
Warning                         : [minor] Trailer data after PNG IEND chunk
```

Some Googling tells us that this PNG has some extra data after the IEND image trailer. This is a warning because the IEND image traler should be the end of the data, according to the PNG image spec.

> 4.1.4. IEND Image trailer
> The IEND chunk must appear LAST. It marks the end of the PNG datastream. The chunk's data field is empty. 
> [source](https://www.w3.org/TR/PNG-Chunks.html)

If we run `strings`, we can see if there's any human-readable data.

```shell
$ strings dolls.jpg
...snip...
?41v57
4wed34e
` 'h
pR}tt
base_images/2_c.jpgUT
O`ux
```

Attempting to download a file called `base_images/2_c.jpg` from the same server as the `dolls.jpg` yeilded no results.

If we upload the file to https://www.nayuki.io/page/png-file-chunk-inspector to inspect the chucks, looking particularly at the data after IEND, we get this.

```
Start Offset
272 492


Raw Bytes
50 4b 03 04 14 00 00 00 08 00 e2 02 70 52 7d 74 74 d8 4c c8 05 00 c2 db 05 00 13 00 1c 00 62 61 73 65 5f 69 6d 61 67 65 73 2f 32 5f 63 2e 6a 70 67 55 54 09 00 03 68 fa 4f 60 68 fa 4f 60 75 78 0b 00 01 04 00 00 ... 00 00 00 04 00 00 00 00 50 4b 05 06 00 00 00 00 01 00 01 00 59 00 00 00 99 c8 05 00 00 00	

Chunk Outside
Data length: 1 347 093 252 bytes
Type: ␔␀␀␀
Name: Unknown
Critical (0)
Public (0)
Reserved (0)
Unsafe to copy (0)
CRC-32: Unfinished

Errors
Premature EOF
Type contains non-alphabetic characters
Chunk must be before IEND chunk
```

If we take the raw bytes and put them into https://gchq.github.io/CyberChef/, and use From Hex, we get a familiar output.

```
...base_images/2_c.jpg...
```

After which, CyberChef's magic wand offers an observation.

> PKZIP archive file dectected

Let's try to `unzip`.

```shell
$ unzip dolls.jpg
warning [dolls.jpg]:  272492 extra bytes at beginning or within zipfile
  (attempting to process anyway)
  inflating: base_images/2_c.jpg 
```

Here, the warning tells us that there are extra bytes at the begin of the file, which tracks with our earlier observation that there are extra bytes at the end of the PNG file.

`xiftool` gives us the same warning on our new file, perhaps it's also a zip.

```shell
$ exiftool base_images/2_c.jpg | grep Warning
Warning                         : [minor] Trailer data after PNG IEND chunk
```

Let's `unzip` again.

```shell
$ unzip base_images/2_c.jpg
Archive:  base_images/2_c.jpg
warning [base_images/2_c.jpg]:  187707 extra bytes at beginning or within zipfile
  (attempting to process anyway)
  inflating: base_images/3_c.jpg
```

Again and again until...

```shell
$ exiftool base_images/3_c.jpg | grep Warning
Warning                         : [minor] Trailer data after PNG IEND chunk
thomascountz-picoctf@webshell:~/matryoshka_doll$ unzip base_images/3_c.jpg
Archive:  base_images/3_c.jpg
warning [base_images/3_c.jpg]:  123606 extra bytes at beginning or within zipfile
  (attempting to process anyway)
  inflating: base_images/4_c.jpg     
thomascountz-picoctf@webshell:~/matryoshka_doll$ exiftool base_images/4_c.jpg | grep Warning
Warning                         : [minor] Trailer data after PNG IEND chunk
thomascountz-picoctf@webshell:~/matryoshka_doll$ unzip base_images/4_c.jpg
Archive:  base_images/4_c.jpg
warning [base_images/4_c.jpg]:  79578 extra bytes at beginning or within zipfile
  (attempting to process anyway)
  inflating: flag.txt
```

Now we have a flag.

```shell
$ cat flag.txt           
picoCTF{336cf6d51c9d9774fd37196c1d7320ff}
```
