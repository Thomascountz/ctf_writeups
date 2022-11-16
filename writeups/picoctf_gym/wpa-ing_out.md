---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: [aircrack-ng]
url: https://play.picoctf.org/practice/challenge/237
points: 200
captured: 2022-11-16
flag: picoCTF{mickeymouse}
summary: Run `aircrack-ng` on the provided PCAP file with the `rockyou.txt` wordlist to crack the WPA password to construct the flag.
---

> I thought that my password was super-secret, but it turns out that passwords passed over the AIR can be CRACKED, especially if I used the same wireless network password as one in the rockyou.txt credential dump. Use this 'pcap file' and the rockyou wordlist.


Before attempting this challenge, I did a bit of reading on WPA, or WiFi Protected Access, networks and how connections are authenticated.

> The four-way handshake was designed to occur over an insecure channel using plaintext, but still provide a means of authenticating and initializing a secure connection between two devices. At no time is any key actually transmitted over the air. The pre-shared key (PSK) is first converted to a primary master key (PMK), which is then used to create the primary transient key (PTK). The PTK is broken down into several parts, one of which is the MIC (Message Authentication Code) Key. This value is then used to create a message digest value (hash) that is appended to each packet for validation. Note that a hash, by definition, cannot be used to re-create the original data. As a result, at no time is sensitive data exposed to an attacker.[^1]

The only way to crack the PSK (pre-shared key), is through brute-force by testing it using the packets passed during the four-way handshake.

If given a PCAP file and a dictionary, `aircrack-ng` can detect this four-way handshake brute-force the shared key. [^2]

```shell
$ aircrack-ng -w /usr/share/wordlists/rockyou.txt wpa-ing_out.pcap 
Reading packets, please wait...
Opening wpa-ing_out.pcap
Read 23523 packets.

   #  BSSID              ESSID                     Encryption

   1  00:5F:67:4F:6A:1A  Gone_Surfing              WPA (1 handshake)

Choosing first network as target.

Reading packets, please wait...
Opening wpa-ing_out.pcap
Read 23523 packets.

1 potential targets



                               Aircrack-ng 1.6 

      [00:00:00] 891/10303727 keys tested (2717.14 k/s) 

      Time left: 1 hour, 3 minutes, 11 seconds                   0.01%

                          KEY FOUND! [ mickeymouse ]


      Master Key     : 61 64 B9 5E FC 6F 41 70 70 81 F6 40 80 9F AF B1 
                       4A 9E C5 C4 E1 67 B8 AB 58 E3 E8 8E E6 66 EB 11 

      Transient Key  : 62 37 2F 54 3B 7B B4 43 9B 37 F4 57 40 FD D1 91 
                       86 7F FE 26 85 7B AC DD 2C 44 E6 06 18 03 B0 0F 
                       F2 75 A2 32 63 F7 35 74 2D 18 10 1C 25 F9 14 BC 
                       41 DA 58 52 48 86 B0 D6 14 89 F6 77 00 8E F7 EB 

      EAPOL HMAC     : 65 2F 6C 0E 75 F0 49 27 6A AA 6A 06 A7 24 B9 A9 

```

Luckily our key, `mickeymouse`, is 20745 out of 14344392 words in `rockyou.txt`, so we get the flag very quickly.

[^1]: https://www.informit.com/articles/article.aspx?p=370636
[^2]: https://www.aircrack-ng.org/doku.php?id=cracking_wpa