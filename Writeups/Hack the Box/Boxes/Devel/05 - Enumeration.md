# Enumeration

## nmap

I started with a standard `nmap` scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/devel]
└─$ nmap -sC -sV -v -oA nmap/devel 10.10.10.5
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-14 11:45 BST
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 11:45
Completed NSE at 11:45, 0.00s elapsed
Initiating NSE at 11:45
Completed NSE at 11:45, 0.00s elapsed
Initiating NSE at 11:45
Completed NSE at 11:45, 0.00s elapsed
Initiating Ping Scan at 11:45
Scanning 10.10.10.5 [2 ports]
Completed Ping Scan at 11:45, 0.02s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 11:45
Completed Parallel DNS resolution of 1 host. at 11:45, 0.00s elapsed
Initiating Connect Scan at 11:45
Scanning 10.10.10.5 [1000 ports]
Discovered open port 21/tcp on 10.10.10.5
Discovered open port 80/tcp on 10.10.10.5
Completed Connect Scan at 11:45, 4.48s elapsed (1000 total ports)
Initiating Service scan at 11:45
Scanning 2 services on 10.10.10.5
Completed Service scan at 11:45, 6.18s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.10.5.
Initiating NSE at 11:45
NSE: [ftp-bounce] PORT response: 501 Server cannot accept argument.
Completed NSE at 11:45, 1.32s elapsed
Initiating NSE at 11:45
Completed NSE at 11:45, 0.29s elapsed
Initiating NSE at 11:45
Completed NSE at 11:45, 0.00s elapsed
Nmap scan report for 10.10.10.5
Host is up (0.024s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

NSE: Script Post-scanning.
Initiating NSE at 11:45
Completed NSE at 11:45, 0.00s elapsed
Initiating NSE at 11:45
Completed NSE at 11:45, 0.00s elapsed
Initiating NSE at 11:45
Completed NSE at 11:45, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.28 seconds
```

Key findings:
- Windows Box
- Running FTP on port 21
	- Anonymous login is allowed
- Running a webserver on port 80
	- IIS 7.5

### All Ports

I waited a second to make sure the box wasn't blocking ping probes, then set off an all ports scan to run after my main one finished:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/devel]
└─$ sleep 300; nmap -p- -oA nmap/devel-allports 10.10.10.5
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-14 11:50 BST
Nmap scan report for 10.10.10.5
Host is up (0.021s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 131.92 seconds
```

It didn't find any new ports.