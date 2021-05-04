# Enumeration

I started out with an `nmap`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB]
└─$ mkdir jerry && cd jerry && mkdir nmap
┌──(mac㉿kali)-[~/Documents/HTB/jerry]
└─$ nmap -sC -sV -oA nmap/ 10.10.10.95
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-04 12:35 BST
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 4.09 seconds
┌──(mac㉿kali)-[~/Documents/HTB/jerry]
└─$ nmap -v -sC -sV -Pn -oA nmap/ 10.10.10.95
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-04 12:35 BST
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 12:35
Completed NSE at 12:35, 0.00s elapsed
Initiating NSE at 12:35
Completed NSE at 12:35, 0.00s elapsed
Initiating NSE at 12:35
Completed NSE at 12:35, 0.00s elapsed
Initiating Parallel DNS resolution of 1 host. at 12:35
Completed Parallel DNS resolution of 1 host. at 12:35, 0.01s elapsed
Initiating Connect Scan at 12:35
Scanning 10.10.10.95 [1000 ports]
Discovered open port 8080/tcp on 10.10.10.95
Completed Connect Scan at 12:35, 7.38s elapsed (1000 total ports)
Initiating Service scan at 12:35
Scanning 1 service on 10.10.10.95
Completed Service scan at 12:35, 6.23s elapsed (1 service on 1 host)
NSE: Script scanning 10.10.10.95.
Initiating NSE at 12:35
Completed NSE at 12:35, 0.81s elapsed
Initiating NSE at 12:35
Completed NSE at 12:35, 0.10s elapsed
Initiating NSE at 12:35
Completed NSE at 12:35, 0.00s elapsed
Nmap scan report for 10.10.10.95
Host is up (0.028s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88

NSE: Script Post-scanning.
Initiating NSE at 12:35
Completed NSE at 12:35, 0.00s elapsed
Initiating NSE at 12:35
Completed NSE at 12:35, 0.00s elapsed
Initiating NSE at 12:35
Completed NSE at 12:35, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.41 seconds
```

This shows just one port open, an Apache Tomcat server on port 8080. It is running version 7.0.88.

### All Ports Scan

This time I checked whether the `-Pn` flag was needed before setting off my all ports scan. It is shown below:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/jerry]
└─$ sleep 300; nmap -Pn -p- -oA nmap/all-ports 10.10.10.95
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-04 12:41 BST
Nmap scan report for 10.10.10.95
Host is up (0.023s latency).
Not shown: 65534 filtered ports
PORT     STATE SERVICE
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 109.57 seconds
```

It found no extra ports.

### Vuln Scan

I ran a vuln scan on the 8080 port:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/jerry]
└─$ nmap --script vuln -Pn -p 8080 -oA nmap/vuln 10.10.10.95
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-04 12:38 BST
Nmap scan report for 10.10.10.95
Host is up (0.021s latency).

PORT     STATE SERVICE
8080/tcp open  http-proxy
| http-enum: 
|   /examples/: Sample scripts
|   /manager/html/upload: Apache Tomcat (401 Unauthorized)
|   /manager/html: Apache Tomcat (401 Unauthorized)
|_  /docs/: Potentially interesting folder
| http-slowloris-check: 
|   VULNERABLE:
|   Slowloris DOS attack
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|       Slowloris tries to keep many connections to the target web server open and hold
|       them open as long as possible.  It accomplishes this by opening connections to
|       the target web server and sending a partial request. By doing so, it starves
|       the http server's resources causing Denial Of Service.
|       
|     Disclosure date: 2009-09-17
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
|_      http://ha.ckers.org/slowloris/

Nmap done: 1 IP address (1 host up) scanned in 97.73 seconds
```

This shows it is vulnerable to slow loris, but we are not trying to crash the box, so this is unlikely to be useful.

## Gobuster

`nmap`'s vuln scan did find some default files, so I decided to set off a `gobuster` scan while I poked at the Tomcat version.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/jerry]
└─$ gobuster dir -u http://10.10.10.95:8080 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.95:8080
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/05/04 12:44:29 Starting gobuster in directory enumeration mode
===============================================================
/docs                 (Status: 302) [Size: 0] [--> /docs/]
/manager              (Status: 302) [Size: 0] [--> /manager/]
/.                    (Status: 200) [Size: 11398]            
/examples             (Status: 302) [Size: 0] [--> /examples/]
/shell                (Status: 302) [Size: 0] [--> /shell/]   
/con                  (Status: 200) [Size: 0]                 
                                                              
===============================================================
2021/05/04 12:46:13 Finished
===============================================================
```

This came back with a few interesting results, including a `shell` directory.

## Enumerating OS

This came later in the box, when I realised I was unsure about the box's Operating System.

We can look at the OS with a simple `ping`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/jerry]
└─$ ping 10.10.10.95
PING 10.10.10.95 (10.10.10.95) 56(84) bytes of data.
64 bytes from 10.10.10.95: icmp_seq=1 ttl=127 time=41.0 ms
64 bytes from 10.10.10.95: icmp_seq=2 ttl=127 time=20.8 ms
64 bytes from 10.10.10.95: icmp_seq=3 ttl=127 time=21.2 ms
^C
--- 10.10.10.95 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 20.803/27.659/41.006/9.438 ms
```

Inspecting the TTL tells us a bit about the OS. The default windows TTL is 128, and `ping` decrements by 1, so this is likely a windows box. For Linux, it is usually 64.

We can also run an nmap OS discovery to check this - the scan requires root privileges:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/jerry]
└─$ sudo nmap -O -Pn -oA nmap/os 10.10.10.95
[sudo] password for mac: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-04 13:00 BST
Nmap scan report for 10.10.10.95
Host is up (0.026s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE
8080/tcp open  http-proxy
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2012 (91%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows 7 Professional (87%), Microsoft Windows 8.1 Update 1 (86%), Microsoft Windows Phone 7.5 or 8.0 (86%), Microsoft Windows 7 or Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 or Windows 8.1 (85%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (85%)
No exact OS matches for host (test conditions non-ideal).

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.51 seconds
```

I don't often use the `-O` flag, as when there are more ports than just 8080 open it is often easier to tell what the OS is.

# Tags

#writeup #oscp-prep #enum 