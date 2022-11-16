---
ctf: picoctf_gym
competition: false
categories: [forensics]
tools: []
url: https://play.picoctf.org/practice/challenge/206
points: 200
captured: 2022-11-13
flag: 
summary: 
---

> I sent my secret flag over the wires, but the bytes got all mixed up!

We get two files, `send.py` and `capture.pcapng`.

Opening up the `send.py` file, we find a `main` method that takes in arguments, parses a payload file into bytes, shuffles them, and then `XOR`-s them one at a time with a random integer between `0` and `256` before sending them (one at a time) to a specified IP and port via UDP.

```python
def main(args):
  with open(args.input, 'rb') as f:
    payload = bytearray(f.read())
  random.seed(int(time()))
  random.shuffle(payload)
  with IncrementalBar('Sending', max=len(payload)) as bar:
    for b in payload:
      send(
        IP(dst=str(args.destination)) /
        UDP(sport=random.randrange(65536), dport=args.port) /
        Raw(load=bytes([b^random.randrange(256)])),
      verbose=False)
      bar.next()
```

To send the bytes, they're using a library called `scapy`, which is a "...powerful Python-based interactive packet manipulation program and library." [^1] 

Also of note, they're supplying a `seed` to python's `random` module set to `int(time)`. This means that if we know the time at which this code was executed, we can replicate the `shuffle` and `randrange` output.

If we open the `capture.pcapng` file, we can see a series of 1-byte UDP packets to `172.17.0.3` on port `56742`. 

We can see what time the first UDP packet was sent in order to get our `seed`.

```
1614044650.913789387
```

---

This is as far as I got with this one. My python skills aren't sharp enough to be able to tackle this one without spending an unreasonable amount of time.

That said, the next steps would be to 1) collect the bytes from the `pcapng`, 2) undo the XOR, 3) undo the shuffle, 4) save bytes as a file.

Here's a great writeup here: https://activities.tjhsst.edu/csc/writeups/picomini-redpwn-darin

[^1]: https://pypi.org/project/scapy/