# Creating a Hashfile

## Adding a Hash

```bash
echo -n HASH >> hashes
```

The `-n` flag removes the trailing newline

## Remove Trailing Newlines

**Remove One:**

```bash
truncate -s $(($(stat -c '%s' hashes)-1)) hashes
```

**Remove All:**

```bash
sed -i 's/$//' hashes
```

# Tags

#cheat-sheet #cryptography 