---
ctf: picoctf
competition: false
categories: forensics
tools: wireshark
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

