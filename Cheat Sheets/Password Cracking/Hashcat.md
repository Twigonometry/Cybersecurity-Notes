# Hashcat

Excellent tutorial available [here](https://resources.infosecinstitute.com/topic/hashcat-tutorial-beginners/)

It is advised to crack on your host machine, not on a VM. Installing Windows Subsytem for Linux can be good for this if you have a windows host machine. Cloud computing can also be useful, although expensive.

## Basic Syntax

```bash
hashcat -m 0 -a 0 -o cracked hashes /usr/share/wordlists/rockyou.txt
```

Where `-o` designates output file, `-a 0` designates dictionary mode, and `-m 0` is mode 0 (MD5)

Hashcat will give you output similar to the below, with either `Status: Cracked` (success) or `Status: Exhausted` (failure):

```bash
HASH:CRACKED_PASS
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: HASH_TYPE
Hash.Target......: HASH
Time.Started.....: Sat Apr  3 14:55:24 2021 (8 secs)
Time.Estimated...: Sat Apr  3 14:55:32 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:       46 H/s (10.53ms) @ Accel:32 Loops:1024 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests
Progress.........: 256/14344385 (0.00%)
Rejected.........: 0/256 (0.00%)
Restore.Point....: 224/14344385 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:31744-32768
Candidates.#1....: tiffany -> freedom
```

If you get a `Separator unmatched` error, this probably means hashcat is expecting some sort of separator character (usually a colon `:`) to be somewhere in your hash. This means you have picked an invalid mode, and need to choose a different one.

## Choosing Mode

Run this command to see hashcat's list of example hashes.

```bash
$ hashcat --example-hashes
```

This list is also available [here](https://hashcat.net/wiki/doku.php?id=example_hashes)

You can search the list with `grep` to find hashes that are similar to yours:

```bash
$ hashcat --example-hashes | grep '[PORTION OF HASH]' -B 3
```

The `-B 3` flag shows the previous 3 lines before the match. This gives us the details of the `MODE` number and the algorithm name.

**Add Notes on using regexes here.**

See the [[Identifying Hash]] cheat sheet for more techniques.