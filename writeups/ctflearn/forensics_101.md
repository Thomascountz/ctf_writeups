---
ctf: ctflearn
competition: false
categories: [forensics]
tools: [strings]
url: https://ctflearn.com/challenge/96
points: 30
captured: 2022-11-03
flag: flag{wow!_data_is_cool}
summary: Find a flag carved into a JPG file using strings.
---

> Think the flag is somewhere in there. Would you help me find it? https://mega.nz/#!OHohCbTa!wbg60PARf4u6E6juuvK9-aDRe_bgEL937VO01EImM7c

```shell
$ strings 95f6edfb66ef42d774a5a34581f19052.jpg | grep flag
flag{wow!_data_is_cool}
```