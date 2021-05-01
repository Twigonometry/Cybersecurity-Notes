# Enumeration

## nmap

I started off with an `nmap` scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ nmap -v -sC -sV -oA nmap/ 10.10.10.40
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-01 11:53 BST
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 11:53
Completed NSE at 11:53, 0.00s elapsed
Initiating NSE at 11:53
Completed NSE at 11:53, 0.00s elapsed
Initiating NSE at 11:53
Completed NSE at 11:53, 0.00s elapsed
Initiating Ping Scan at 11:53
Scanning 10.10.10.40 [2 ports]
Completed Ping Scan at 11:53, 0.04s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 11:53
Completed Parallel DNS resolution of 1 host. at 11:53, 0.00s elapsed
Initiating Connect Scan at 11:53
Scanning 10.10.10.40 [1000 ports]
Discovered open port 135/tcp on 10.10.10.40
Discovered open port 445/tcp on 10.10.10.40
Discovered open port 139/tcp on 10.10.10.40
Discovered open port 49156/tcp on 10.10.10.40
Discovered open port 49154/tcp on 10.10.10.40
Discovered open port 49153/tcp on 10.10.10.40
Discovered open port 49152/tcp on 10.10.10.40
Discovered open port 49155/tcp on 10.10.10.40
Discovered open port 49157/tcp on 10.10.10.40
Completed Connect Scan at 11:53, 1.59s elapsed (1000 total ports)
Initiating Service scan at 11:53
Scanning 9 services on 10.10.10.40
Service scan Timing: About 44.44% done; ETC: 11:55 (0:01:08 remaining)
Completed Service scan at 11:54, 59.30s elapsed (9 services on 1 host)
NSE: Script scanning 10.10.10.40.
Initiating NSE at 11:54
Completed NSE at 11:54, 10.11s elapsed
Initiating NSE at 11:54
Completed NSE at 11:54, 0.01s elapsed
Initiating NSE at 11:54
Completed NSE at 11:54, 0.00s elapsed
Nmap scan report for 10.10.10.40
Host is up (0.026s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -16m03s, deviation: 34m36s, median: 3m54s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-05-01T11:58:35+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-01T10:58:33
|_  start_date: 2021-05-01T10:55:38

NSE: Script Post-scanning.
Initiating NSE at 11:54
Completed NSE at 11:54, 0.00s elapsed
Initiating NSE at 11:54
Completed NSE at 11:54, 0.00s elapsed
Initiating NSE at 11:54
Completed NSE at 11:54, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.13 seconds
```

This shows a large number of RPC ports, and an SMB service on port 445.

OS Discovery reveals the operating system to be `Windows 7 Professional 7601`, and reveals a potential user `haris`

## Enumerating SMB

I tried a basic `smbmap` against the host, then tried enumerating with null authentication:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ smbmap -H 10.10.10.40
[+] IP: 10.10.10.40:445	Name: 10.10.10.40
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ smbmap -u null -p "" -H 10.10.10.40
[+] Guest session   	IP: 10.10.10.40:445	Name: 10.10.10.40                                       
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	NO ACCESS	Remote IPC
	Share                                             	READ ONLY	
	Users                                             	READ ONLY
```

We can see we have access to the `Share` share and the `Users` share. Let's connect to them and see what's inside:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/blue]
└─$ smbclient //10.10.10.40/Users
Enter WORKGROUP\mac's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Fri Jul 21 07:56:23 2017
  ..                                 DR        0  Fri Jul 21 07:56:23 2017
  Default                           DHR        0  Tue Jul 14 08:07:31 2009
  desktop.ini                       AHS      174  Tue Jul 14 05:54:24 2009
  Public                             DR        0  Tue Apr 12 08:51:29 2011

		8362495 blocks of size 4096. 4258428 blocks available
smb: \> cd Default\
smb: \Default\> dir
  .                                 DHR        0  Tue Jul 14 08:07:31 2009
  ..                                DHR        0  Tue Jul 14 08:07:31 2009
  AppData                           DHn        0  Tue Jul 14 04:20:08 2009
  Desktop                            DR        0  Tue Jul 14 03:34:59 2009
  Documents                          DR        0  Tue Jul 14 06:08:56 2009
  Downloads                          DR        0  Tue Jul 14 03:34:59 2009
  Favorites                          DR        0  Tue Jul 14 03:34:59 2009
  Links                              DR        0  Tue Jul 14 03:34:59 2009
  Music                              DR        0  Tue Jul 14 03:34:59 2009
  NTUSER.DAT                       AHSn   262144  Fri Jul 14 23:37:57 2017
  NTUSER.DAT.LOG                     AH     1024  Tue Apr 12 08:54:55 2011
  NTUSER.DAT.LOG1                    AH   189440  Sun Jul 16 21:22:24 2017
  NTUSER.DAT.LOG2                    AH        0  Tue Jul 14 03:34:08 2009
  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TM.blf    AHS    65536  Tue Jul 14 05:45:54 2009
  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000001.regtrans-ms    AHS   524288  Tue Jul 14 05:45:54 2009
  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000002.regtrans-ms    AHS   524288  Tue Jul 14 05:45:54 2009
  Pictures                           DR        0  Tue Jul 14 03:34:59 2009
  Saved Games                        Dn        0  Tue Jul 14 03:34:59 2009
  Videos                             DR        0  Tue Jul 14 03:34:59 2009

		8362495 blocks of size 4096. 4258428 blocks available
smb: \Default\> 

```

The `Users` share is particularly interesting, and contains some `NTUSER.DAT` files. These are Windows Profile files, and it seems it is potentially possible to [extract password information](https://www.fuzzysecurity.com/tutorials/18.html) from them. We'll make a note of this and potentially come back to these later if we run out of ideas.

### SMB Vulnerability Scan

I didn't do this when I first ran through the box, but the following command would have immediately raised that the box was vulnerable to SMB.

```bash
┌──(mac㉿kali)-[~/Documents/Personal-Vault]
└─$ nmap --script vuln 10.10.10.40
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-01 19:41 BST
Nmap scan report for 10.10.10.40
Host is up (0.032s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown

Host script results:
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_OBJECT_NAME_NOT_FOUND
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/

Nmap done: 1 IP address (1 host up) scanned in 112.68 seconds
```

I found out it was vulnerable a different way, detailed in the next section.