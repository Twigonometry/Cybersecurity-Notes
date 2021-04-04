# Generating a Wordlist

## Using Hashcat

Use a rules file (found in `/usr/share/hashcat/rules`)

### Toggles

```bash
hashcat --force --stdout passwords -r /usr/share/hashcat/rules/toggles1.rule > passwordlist
```

### Best64 Rules
```bash
hashcat --force --stdout passwords -r /usr/share/hashcat/rules/best64.rule > passwordlist
```

## Trimming Wordlist

### Unique Passes

```bash
cat passwordlist | sort -u > passwordlist-unique
```

### By Length

```bash
cat passwordlist | awk 'length($0) > 7' > passwordlist-eight
```

### Check Number

Pipe output to `wc -l` to check how many are in the list