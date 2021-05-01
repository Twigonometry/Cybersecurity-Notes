# Generating a Wordlist

You can generate a custom wordlist for password cracking when you have an idea of what a password might be, and think there may be some variation on a word or phrase.

## Using Hashcat

Use a rules file (found in `/usr/share/hashcat/rules`)

You can chain these rules together to create a larger wordlist. I like starting with 3-4 password ideas specific to the user I am trying to crack, then using `best64.rule` followed by `toggles1.rule` on the result.

[[Hashcat|See more about Hashcat]]

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

# Tags

#cheat-sheet #cryptography 