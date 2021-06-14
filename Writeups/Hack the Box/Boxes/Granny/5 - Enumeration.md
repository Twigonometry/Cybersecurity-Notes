# Enumeration

## nmap

I started with an `nmap`, using the `-v` flag to show me ports as they're discovered.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ nmap -sC -sV -v -oA nmap/ 10.10.10.15
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-06 08:29 BST
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 08:29
Completed NSE at 08:29, 0.00s elapsed
Initiating NSE at 08:29
Completed NSE at 08:29, 0.00s elapsed
Initiating NSE at 08:29
Completed NSE at 08:29, 0.00s elapsed
Initiating Ping Scan at 08:29
Scanning 10.10.10.15 [2 ports]
Completed Ping Scan at 08:29, 0.04s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 08:29
Completed Parallel DNS resolution of 1 host. at 08:29, 0.00s elapsed
Initiating Connect Scan at 08:29
Scanning 10.10.10.15 [1000 ports]
Discovered open port 80/tcp on 10.10.10.15
Completed Connect Scan at 08:29, 8.61s elapsed (1000 total ports)
Initiating Service scan at 08:29
Scanning 1 service on 10.10.10.15
Completed Service scan at 08:29, 6.14s elapsed (1 service on 1 host)
NSE: Script scanning 10.10.10.15.
Initiating NSE at 08:29
Completed NSE at 08:29, 0.59s elapsed
Initiating NSE at 08:29
Completed NSE at 08:29, 0.09s elapsed
Initiating NSE at 08:29
Completed NSE at 08:29, 0.00s elapsed
Nmap scan report for 10.10.10.15
Host is up (0.030s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT POST
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|   Server Date: Thu, 06 May 2021 07:33:20 GMT
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   WebDAV type: Unknown
|_  Server Type: Microsoft-IIS/6.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

NSE: Script Post-scanning.
Initiating NSE at 08:29
Completed NSE at 08:29, 0.00s elapsed
Initiating NSE at 08:29
Completed NSE at 08:29, 0.00s elapsed
Initiating NSE at 08:29
Completed NSE at 08:29, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.99 seconds
```

The only port open was 80, showing a webserver running Microsoft-IIS/6.0. This is a windows box.

Crucially, it also showed that WebDAV is running.

### All Ports Scan

I ran an all ports scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ nmap -p- -oA nmap/all-ports 10.10.10.15
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-06 08:30 BST
Nmap scan report for 10.10.10.15
Host is up (0.021s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 106.82 seconds
```

It found no new ports.

### UDP Scan

I also ran a UDP scan, which found nothing:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ sudo nmap -sU -oA nmap/udp 10.10.10.15
[sudo] password for mac: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-06 08:35 BST
Nmap scan report for 10.10.10.15
Host is up (0.020s latency).
All 1000 scanned ports on 10.10.10.15 are open|filtered

Nmap done: 1 IP address (1 host up) scanned in 21.34 seconds
```

## Gobuster

I ran an initial gobuster:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ gobuster dir -u http://10.10.10.15 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.15
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/05/06 08:35:43 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 149] [--> http://10.10.10.15/images/]
/_private             (Status: 301) [Size: 153] [--> http://10.10.10.15/%5Fprivate/]
/aspnet_client        (Status: 301) [Size: 158] [--> http://10.10.10.15/aspnet%5Fclient/]
/_vti_log             (Status: 301) [Size: 155] [--> http://10.10.10.15/%5Fvti%5Flog/]   
/_vti_bin             (Status: 301) [Size: 155] [--> http://10.10.10.15/%5Fvti%5Fbin/]   
/Images               (Status: 301) [Size: 149] [--> http://10.10.10.15/Images/]         
/.                    (Status: 200) [Size: 1433]                                         
/IMAGES               (Status: 301) [Size: 149] [--> http://10.10.10.15/IMAGES/]         
/Aspnet_client        (Status: 301) [Size: 158] [--> http://10.10.10.15/Aspnet%5Fclient/]
/_Private             (Status: 301) [Size: 153] [--> http://10.10.10.15/%5FPrivate/]     
/aspnet_Client        (Status: 301) [Size: 158] [--> http://10.10.10.15/aspnet%5FClient/]
/ASPNET_CLIENT        (Status: 301) [Size: 158] [--> http://10.10.10.15/ASPNET%5FCLIENT/]
                                                                                         
===============================================================
2021/05/06 08:37:20 Finished
===============================================================
```

### Extra Gobuster

I did a Gobuster search specifically for directories:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ gobuster dir -u http://10.10.10.15 -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.15
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/05/06 08:50:52 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 149] [--> http://10.10.10.15/images/]
/aspnet_client        (Status: 301) [Size: 158] [--> http://10.10.10.15/aspnet%5Fclient/]
/_private             (Status: 301) [Size: 153] [--> http://10.10.10.15/%5Fprivate/]     
/_vti_log             (Status: 301) [Size: 155] [--> http://10.10.10.15/%5Fvti%5Flog/]   
/_vti_bin             (Status: 301) [Size: 155] [--> http://10.10.10.15/%5Fvti%5Fbin/]   
/Images               (Status: 301) [Size: 149] [--> http://10.10.10.15/Images/]         
/IMAGES               (Status: 301) [Size: 149] [--> http://10.10.10.15/IMAGES/]         
/Aspnet_client        (Status: 301) [Size: 158] [--> http://10.10.10.15/Aspnet%5Fclient/]
/_Private             (Status: 301) [Size: 153] [--> http://10.10.10.15/%5FPrivate/]     
/aspnet_Client        (Status: 301) [Size: 158] [--> http://10.10.10.15/aspnet%5FClient/]
                                                                                         
===============================================================
2021/05/06 08:51:48 Finished
===============================================================
```

It found no new ones.

I also did a Gobuster scan with a larger wordlist:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ gobuster dir -u http://10.10.10.15 -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.15
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/05/06 08:52:26 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 149] [--> http://10.10.10.15/images/]
/_private             (Status: 301) [Size: 153] [--> http://10.10.10.15/%5Fprivate/]
/aspnet_client        (Status: 301) [Size: 158] [--> http://10.10.10.15/aspnet%5Fclient/]
/_vti_log             (Status: 301) [Size: 155] [--> http://10.10.10.15/%5Fvti%5Flog/]   
/_vti_bin             (Status: 301) [Size: 155] [--> http://10.10.10.15/%5Fvti%5Fbin/]   
/Images               (Status: 301) [Size: 149] [--> http://10.10.10.15/Images/]         
/.                    (Status: 200) [Size: 1433]                                         
/IMAGES               (Status: 301) [Size: 149] [--> http://10.10.10.15/IMAGES/]         
/Aspnet_client        (Status: 301) [Size: 158] [--> http://10.10.10.15/Aspnet%5Fclient/]
/_Private             (Status: 301) [Size: 153] [--> http://10.10.10.15/%5FPrivate/]     
/aspnet_Client        (Status: 301) [Size: 158] [--> http://10.10.10.15/aspnet%5FClient/]
/ASPNET_CLIENT        (Status: 301) [Size: 158] [--> http://10.10.10.15/ASPNET%5FCLIENT/]
/_PRIVATE             (Status: 301) [Size: 153] [--> http://10.10.10.15/%5FPRIVATE/]     
/_VTI_LOG             (Status: 301) [Size: 155] [--> http://10.10.10.15/%5FVTI%5FLOG/]   
/Aspnet_Client        (Status: 301) [Size: 158] [--> http://10.10.10.15/Aspnet%5FClient/]
                                                                                         
===============================================================
2021/05/06 08:57:11 Finished
===============================================================
```

This also found no useful results.