---
ctf: picoctf
competition: false
categories: forensics
url: https://play.picoctf.org/practice/challenge/115
captured: 2022-10-25
flag:
---

# Wireshark doo dooo do doo...

> Can you find the flag? shark1.pcapng.

PcapNg (the file extension) is a PCAP Next Generation Dump File Format to overcome the limitations of the libpcap format. [source](https://wiki.wireshark.org/Development/PcapNg)

PcapNg can immediately be opened in wireshark.

![](shark1pcapng_in_wireshark.png)
Other CTF writeups have suggested to begin Wireshark investigations with an HTTP filter.

```
http.response.code == 200
```

After doing so, we can see that all but two are `(application/http-kerberos-session-encrypted)`. Apparently, "Kerberos (/ˈkɜːrbərɒs/) is a computer-network authentication protocol that works on the basis of tickets to allow nodes communicating over a non-secure network to prove their identity to one another in a secure manner. Its designers aimed it primarily at a client–server model, and it provides mutual authentication—both the user and the server verify each other's identity." And most importantly: "...Kerberos protocol messages are protected against eavesdropping and replay attacks." [source](https://en.wikipedia.org/wiki/Kerberos_(protocol)).

So ignoring those for now, the other two streams are `(text/html)` and `(text/plain)`.

![](shark1pcapng_in_wireshark_text_requests.png)
To view these streams as the application would, we can use Wireshark's [follow](https://www.wireshark.org/docs/wsug_html_chunked/ChAdvFollowStreamSection.html) feature. 

Doing so gives us a more familiar view of the HTTP request and response.

![](shark1pcapng_http_stream_5.png)
Following the first `(text/html)` stream, we get this view.

The response body contains a _somewhat_ familiar-looking string. Similar to the `picoCTF{flag}` format of the CTF flag.

Let's put this in CyberChef.

