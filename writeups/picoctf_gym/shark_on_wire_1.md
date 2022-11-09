---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: [wireshark]
url: https://play.picoctf.org/practice/challenge/30
points: 150
captured: 2022-11-09
flag: picoCTF{StaT31355_636f6e6e}
summary: Use Wireshark to find and follow the UDP stream where the flag was sent one byte at a time.
---

Opening up the `.pcap` file in Wireshark, the first thing we can do is have a look at the Protocol Hierarchy, which tells us the proportion of each protocol that shows up in the capture.

![shark_on_wire_1_hierarchy](./attachments/shark_on_wire_1_hierarchy.png)

At 32.3%, UDP Data packets are the majority. This is also interesting because these data packets will be in cleartext.

If we filter the capture by `data` (which is what happened when I "Right Click->Apply as Filter->Selected" in the Protocol Hierarchy window, but we could just as easily filter by `udp` manually), we see a series packets that appear to be a recurring pattern of 24 bytes, followed by a series of packets of one byte.

When we take a look at the first one byte packet, we see a `p`.

![shark_on_wire_1_udp_filter](./attachments/shark_on_wire_1_udp_filter.png)

If we follow this UDP stream (Right Click->Follow->UDP Stream), we get the flag.

![shark_on_wire_1_udp_stream_6](./attachments/shark_on_wire_1_udp_stream_6.png)