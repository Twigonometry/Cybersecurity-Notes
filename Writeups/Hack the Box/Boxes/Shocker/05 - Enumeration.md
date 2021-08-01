# Enumeration

## Nmap

I started with an `nmap` scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ nmap -sC -sV -v -Pn -oA nmap/shocker 10.10.10.56
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-14 08:22 BST
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 08:22
Completed NSE at 08:22, 0.00s elapsed
Initiating NSE at 08:22
Completed NSE at 08:22, 0.00s elapsed
Initiating NSE at 08:22
Completed NSE at 08:22, 0.00s elapsed
Initiating Parallel DNS resolution of 1 host. at 08:22
Completed Parallel DNS resolution of 1 host. at 08:22, 0.00s elapsed
Initiating Connect Scan at 08:22
Scanning 10.10.10.56 [1000 ports]
Discovered open port 80/tcp on 10.10.10.56
Discovered open port 2222/tcp on 10.10.10.56
Completed Connect Scan at 08:22, 0.46s elapsed (1000 total ports)
Initiating Service scan at 08:22
Scanning 2 services on 10.10.10.56
Completed Service scan at 08:22, 6.12s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.10.56.
Initiating NSE at 08:22
Completed NSE at 08:22, 1.53s elapsed
Initiating NSE at 08:22
Completed NSE at 08:22, 0.33s elapsed
Initiating NSE at 08:22
Completed NSE at 08:22, 0.00s elapsed
Nmap scan report for 10.10.10.56
Host is up (0.036s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 08:22
Completed NSE at 08:22, 0.00s elapsed
Initiating NSE at 08:22
Completed NSE at 08:22, 0.00s elapsed
Initiating NSE at 08:22
Completed NSE at 08:22, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.79 seconds

```

Key findings:
- webserver running on port 80, with Apache 2.4.18
- SSH running on non-standard port 2222
- Ubuntu box

### All ports

I also did a full port scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ sleep 300; nmap -p- -Pn -oA nmap/shocker-allports 10.10.10.56
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-14 08:33 BST
Nmap scan report for 10.10.10.56
Host is up (0.030s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE
80/tcp   open  http
2222/tcp open  EtherNetIP-1

Nmap done: 1 IP address (1 host up) scanned in 13.52 seconds
```

### UDP

And a UDP scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ sudo nmap -sU -Pn -oA nmap/shocker-udp 10.10.10.56
[sudo] password for mac: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-14 08:40 BST
Nmap scan report for 10.10.10.56
Host is up (0.022s latency).
All 1000 scanned ports on 10.10.10.56 are closed

Nmap done: 1 IP address (1 host up) scanned in 1006.91 seconds
```

## Vuln Checks

I ran `nmap`'s `vuln` script to see if there were any quick wins:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ nmap --script vuln -Pn -oA nmap/shocker-vuln 10.10.10.56
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-14 08:27 BST
Nmap scan report for 10.10.10.56
Host is up (0.043s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE
80/tcp   open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
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
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
2222/tcp open  EtherNetIP-1

Nmap done: 1 IP address (1 host up) scanned in 323.34 seconds
```

The vuln check identified port 2222 as running EtherNetIP-1, not SSH, so I ran a quick searchsploit:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ searchsploit ethernetip
Exploits: No Results
Shellcodes: No Results
Papers: No Results
```

But it found nothing.

## Gobuster

I ran a `gobuster` scan on the site:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ gobuster dir -u http://10.10.10.56 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/14 08:31:09 Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 291]
/.htm                 (Status: 403) [Size: 290]
/.                    (Status: 200) [Size: 137]
/.htaccess            (Status: 403) [Size: 295]
/.htc                 (Status: 403) [Size: 290]
/.html_var_DE         (Status: 403) [Size: 298]
/server-status        (Status: 403) [Size: 299]
/.htpasswd            (Status: 403) [Size: 295]
/.html.               (Status: 403) [Size: 292]
/.html.html           (Status: 403) [Size: 296]
/.htpasswds           (Status: 403) [Size: 296]
/.htm.                (Status: 403) [Size: 291]
/.htmll               (Status: 403) [Size: 292]
/.html.old            (Status: 403) [Size: 295]
/.ht                  (Status: 403) [Size: 289]
/.html.bak            (Status: 403) [Size: 295]
/.htm.htm             (Status: 403) [Size: 294]
/.htgroup             (Status: 403) [Size: 294]
/.hta                 (Status: 403) [Size: 290]
/.html1               (Status: 403) [Size: 292]
/.html.printable      (Status: 403) [Size: 301]
/.html.LCK            (Status: 403) [Size: 295]
/.htm.LCK             (Status: 403) [Size: 294]
/.html.php            (Status: 403) [Size: 295]
/.htaccess.bak        (Status: 403) [Size: 299]
/.htmls               (Status: 403) [Size: 292]
/.htx                 (Status: 403) [Size: 290]
/.htlm                (Status: 403) [Size: 291]
/.htuser              (Status: 403) [Size: 293]
/.html-               (Status: 403) [Size: 292]
/.htm2                (Status: 403) [Size: 291]
                                               
===============================================================
2021/06/14 08:33:14 Finished
===============================================================
```

It found nothing - I would end up having to do a few extra `gobuster` scans later on.

## Autorecon

I also launched an autorecon scan after spending some time manually enumerating the webserver and finding nothing, just in case I'd missed something obvious:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ autorecon 10.10.10.56
```

It didn't find any new services. I checked out the `nikto` and `whatweb` scans to see if they raised any new vulnerabilities that I hadn't seen:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker/results/10.10.10.56/scans]
└─$ cat tcp_80_http_nikto.txt
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.56
+ Target Hostname:    10.10.10.56
+ Target Port:        80
+ Start Time:         2021-06-14 09:07:03 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Server may leak inodes via ETags, header found with file /, inode: 89, size: 559ccac257884, mtime: gzip
+ Allowed HTTP Methods: OPTIONS, GET, HEAD, POST 
+ OSVDB-3233: /icons/README: Apache default file found.
+ 8674 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2021-06-14 09:11:13 (GMT1) (250 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
┌──(mac㉿kali)-[~/Documents/HTB/shocker/results/10.10.10.56/scans]
└─$ cat tcp_80_http_whatweb.txt
WhatWeb report for http://10.10.10.56:80
Status    : 200 OK
Title     : <None>
IP        : 10.10.10.56
Country   : RESERVED, ZZ

Summary   : HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], Apache[2.4.18], HTML5

Detected Plugins:
[ Apache ]
	The Apache HTTP Server Project is an effort to develop and 
	maintain an open-source HTTP server for modern operating 
	systems including UNIX and Windows NT. The goal of this 
	project is to provide a secure, efficient and extensible 
	server that provides HTTP services in sync with the current 
	HTTP standards. 

	Version      : 2.4.18 (from HTTP Server Header)
	Google Dorks: (3)
	Website     : http://httpd.apache.org/

[ HTML5 ]
	HTML version 5, detected by the doctype declaration 


[ HTTPServer ]
	HTTP server header string. This plugin also attempts to 
	identify the operating system from the server header. 

	OS           : Ubuntu Linux
	String       : Apache/2.4.18 (Ubuntu) (from server string)

HTTP Headers:
	HTTP/1.1 200 OK
	Date: Mon, 14 Jun 2021 08:14:47 GMT
	Server: Apache/2.4.18 (Ubuntu)
	Last-Modified: Fri, 22 Sep 2017 20:01:19 GMT
	ETag: "89-559ccac257884-gzip"
	Accept-Ranges: bytes
	Vary: Accept-Encoding
	Content-Encoding: gzip
	Content-Length: 134
	Connection: close
	Content-Type: text/html
```

They didn't seem to find anything useful.