# Enumeration

## nmap

I started with an `nmap` scan. I may run autorecon if I run out of ideas, but I'm liking it less and less every time I do it and feel I learn more from manual enum.

Initial nmap:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/optimum]
└─$ nmap -sC -sV -v -oA nmap/optimum 10.10.10.8
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-13 11:07 BST
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 11:07
Completed NSE at 11:07, 0.00s elapsed
Initiating NSE at 11:07
Completed NSE at 11:07, 0.00s elapsed
Initiating NSE at 11:07
Completed NSE at 11:07, 0.00s elapsed
Initiating Ping Scan at 11:07
Scanning 10.10.10.8 [2 ports]
Completed Ping Scan at 11:07, 0.09s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 11:07
Completed Parallel DNS resolution of 1 host. at 11:07, 0.00s elapsed
Initiating Connect Scan at 11:07
Scanning 10.10.10.8 [1000 ports]
Discovered open port 80/tcp on 10.10.10.8
Completed Connect Scan at 11:07, 12.75s elapsed (1000 total ports)
Initiating Service scan at 11:07
Scanning 1 service on 10.10.10.8
Completed Service scan at 11:07, 6.12s elapsed (1 service on 1 host)
NSE: Script scanning 10.10.10.8.
Initiating NSE at 11:07
Completed NSE at 11:07, 1.24s elapsed
Initiating NSE at 11:07
Completed NSE at 11:07, 0.23s elapsed
Initiating NSE at 11:07
Completed NSE at 11:07, 0.00s elapsed
Nmap scan report for 10.10.10.8
Host is up (0.062s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-favicon: Unknown favicon MD5: 759792EDD4EF8E6BC2D1877D27153CB1
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

NSE: Script Post-scanning.
Initiating NSE at 11:07
Completed NSE at 11:07, 0.00s elapsed
Initiating NSE at 11:07
Completed NSE at 11:07, 0.00s elapsed
Initiating NSE at 11:07
Completed NSE at 11:07, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.41 seconds
```

Key findings:
- HTTP server on port 80, running HTTPFileServer 2.3
- Windows Box, not sure about version

That's really it.

### All ports scan

I also ran a quick all ports scan.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/optimum]
└─$ nmap -p- -v -oA nmap/optimum-allports 10.10.10.8
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-13 11:09 BST
Initiating Ping Scan at 11:09
Scanning 10.10.10.8 [2 ports]
Completed Ping Scan at 11:09, 0.02s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 11:09
Completed Parallel DNS resolution of 1 host. at 11:09, 0.01s elapsed
Initiating Connect Scan at 11:09
Scanning 10.10.10.8 [65535 ports]
Discovered open port 80/tcp on 10.10.10.8
Connect Scan Timing: About 17.47% done; ETC: 11:12 (0:02:26 remaining)
Connect Scan Timing: About 33.62% done; ETC: 11:12 (0:02:00 remaining)
Connect Scan Timing: About 53.43% done; ETC: 11:12 (0:01:19 remaining)
Connect Scan Timing: About 68.50% done; ETC: 11:12 (0:00:56 remaining)
Connect Scan Timing: About 82.35% done; ETC: 11:12 (0:00:32 remaining)
Completed Connect Scan at 11:12, 181.08s elapsed (65535 total ports)
Nmap scan report for 10.10.10.8
Host is up (0.036s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 181.49 seconds
```

This found no new ports.

### OS Enum

I tried enumerating the Operating System further with the `-O` flag.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/optimum]
└─$ sudo nmap -O -oA nmap/os 10.10.10.8
[sudo] password for mac: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-13 11:32 BST
Nmap scan report for 10.10.10.8
Host is up (0.022s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
80/tcp open  http
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2012 (91%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows 7 Professional (87%), Microsoft Windows 8.1 Update 1 (86%), Microsoft Windows Phone 7.5 or 8.0 (86%), Microsoft Windows 7 or Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 or Windows 8.1 (85%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (85%)
No exact OS matches for host (test conditions non-ideal).

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.77 seconds
```

The most likely system was Windows Server 2012.

## Gobuster

I ran a quick scan against the root of the website:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/optimum]
└─$ gobuster dir -u http://10.10.10.8 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.8
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/13 11:10:50 Starting gobuster in directory enumeration mode
===============================================================
/.                    (Status: 301) [Size: 44] [--> /]
Progress: 8158 / 43004 (18.97%)                      [ERROR] 2021/06/13 11:12:34 [!] Get "http://10.10.10.8/checkoutpayment": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
                                                      
===============================================================
2021/06/13 11:19:58 Finished
===============================================================
```

It didn't find anything new.