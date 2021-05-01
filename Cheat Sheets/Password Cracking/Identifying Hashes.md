# Identifying Hashes
Run this command to find a list of potential hashing algorithms

```bash
$ hashid [HASH]
```

## Pair with Hashcat

Run these through [[Hashcat]] to see what modes they are for cracking:

```bash
$ hashcat --example-hashes | grep [ALGORITHM-NAME] -B 2
```

If you have a large number of candidates, search them all with this one-liner:

```bash
$ hashid [HASH] | awk '{print $2}' | while read line; do hashcat --example-hashes | grep $line -B 1; done
```

This is a quick and dirty bash script that takes the second column of the `hashid` output (the guessed algorithm) and passes it to hashcat. It could be improved, as it also passes the hash itself to the `grep` command.

# Tags

#cheat-sheet #cryptography 