# Hashcat

Excellent tutorial available [here](https://resources.infosecinstitute.com/topic/hashcat-tutorial-beginners/)

## Basic Syntax

```bash
hashcat -m 0 -a 0 -o cracked hashes /usr/share/wordlists/rockyou.txt
```

Where `-o` designates output file, `-a 0` designates dictionary mode, and `-m 0` is mode 0 (MD5)

## Choosing Mode

```bash
hashcat --example-hashes
```

List also available [here](https://hashcat.net/wiki/doku.php?id=example_hashes)