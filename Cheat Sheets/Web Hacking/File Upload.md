# File Upload

## Magic Bytes

**View magic bytes**
```bash
head -c 20 /path/to/file | xxd
```

**Adding Magic Bytes to File**

Append magic bytes of safe file (e.g. png) to blank unsafe file:
```bash
head -c 8 /path/to/safe/file > unsafe\_file
```

Or concatenate file and stored magic bytes:
```bash
cat /path/to/magic/bytes /path/to/file > new_file
```

# Tags

#cheat-sheet #web