# OpenSSH 

I ran `searchsploit` against openssh (this was after spending some time enumerating the website and getting nowhere, which I'll detail in the next section):

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ searchsploit openssh
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                         |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------

OpenSSH 7.2 - Denial of Service                                                                                                                                        | linux/dos/40888.py
OpenSSH 7.2p1 - (Authenticated) xauth Command Injection                                                                                                                | multiple/remote/39569.py
OpenSSH 7.2p2 - Username Enumeration                                                                                                                                   | linux/remote/40136.py

OpenSSH < 7.4 - 'UsePrivilegeSeparation Disabled' Forwarded Unix Domain Sockets Privilege Escalation                                                                   | linux/local/40962.txt
OpenSSH < 7.4 - agent Protocol Arbitrary Library Loading                                                                                                               | linux/remote/40963.txt
OpenSSH < 7.7 - User Enumeration (2)                                                                                                                                   | linux/remote/45939.py
```

The most promising exploit, `multiple/remote/39569.py`,  requires authentication.

I even tried logging in with no username/key just in case there was some weird SSH misconfiguration:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ ssh -p 2222 10.10.10.56
The authenticity of host '[10.10.10.56]:2222 ([10.10.10.56]:2222)' can't be established.
ECDSA key fingerprint is SHA256:6Xub2G5qowxZGyUBvUK4Y0prznGD5J2UyeMhJSdCZGw.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.10.56]:2222' (ECDSA) to the list of known hosts.
mac@10.10.10.56's password: 
Permission denied, please try again.
mac@10.10.10.56's password: 
Permission denied, please try again.
mac@10.10.10.56's password: 
```

But no luck. It was worth doing our due diligence here, but SSH likely isn't vulnerable.