# Enumeration

## nmap

I did an initial `nmap` scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bashed]
└─$ nmap -sC -sV -v -oA nmap/ 10.10.10.68
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-05 08:29 BST
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 08:29
Completed NSE at 08:29, 0.00s elapsed
Initiating NSE at 08:29
Completed NSE at 08:29, 0.00s elapsed
Initiating NSE at 08:29
Completed NSE at 08:29, 0.00s elapsed
Initiating Ping Scan at 08:29
Scanning 10.10.10.68 [2 ports]
Completed Ping Scan at 08:29, 0.05s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 08:29
Completed Parallel DNS resolution of 1 host. at 08:29, 0.00s elapsed
Initiating Connect Scan at 08:29
Scanning 10.10.10.68 [1000 ports]
Discovered open port 80/tcp on 10.10.10.68
Completed Connect Scan at 08:29, 0.48s elapsed (1000 total ports)
Initiating Service scan at 08:29
Scanning 1 service on 10.10.10.68
Completed Service scan at 08:30, 6.06s elapsed (1 service on 1 host)
NSE: Script scanning 10.10.10.68.
Initiating NSE at 08:30
Completed NSE at 08:30, 0.64s elapsed
Initiating NSE at 08:30
Completed NSE at 08:30, 0.10s elapsed
Initiating NSE at 08:30
Completed NSE at 08:30, 0.00s elapsed
Nmap scan report for 10.10.10.68
Host is up (0.029s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 6AA5034A553DFA77C3B2C7B4C26CF870
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site

NSE: Script Post-scanning.
Initiating NSE at 08:30
Completed NSE at 08:30, 0.00s elapsed
Initiating NSE at 08:30
Completed NSE at 08:30, 0.00s elapsed
Initiating NSE at 08:30
Completed NSE at 08:30, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.82 seconds
```

This shows just a web server, titled "Arrexel's Development SIte" - this exposes a potential user. It is running Apache 2.4.18 on Ubuntu.

### Full Port Scan

As the standard nmap finished so quickly, I cancelled out of the usual `sleep 300` and just ran a full port scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bashed]
└─$ nmap -p- -oA nmap/all-ports 10.10.10.68
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-05 08:33 BST
Nmap scan report for 10.10.10.68
Host is up (0.025s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 442.66 seconds
```

This found nothing new.

### UDP Scan

I also ran a UDP scan, which I don't usually do - but as there was only one port open, I thought it was worth doing. Scanning UDP requires root privileges:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bashed]
└─$ sudo nmap -sU -oA nmap/udp 10.10.10.68
[sudo] password for mac: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-05 09:14 BST
Nmap scan report for 10.10.10.68
Host is up (0.023s latency).
All 1000 scanned ports on 10.10.10.68 are closed

Nmap done: 1 IP address (1 host up) scanned in 1086.88 seconds
```

There was nothing running on UDP.

## Gobuster

Once I found the site I ran a gobuster:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bashed]
└─$ gobuster dir -u http://10.10.10.68 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.68
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/05/05 09:04:26 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 290]
/images               (Status: 301) [Size: 311] [--> http://10.10.10.68/images/]
/.html                (Status: 403) [Size: 291]                                 
/js                   (Status: 301) [Size: 307] [--> http://10.10.10.68/js/]    
/css                  (Status: 301) [Size: 308] [--> http://10.10.10.68/css/]   
/.htm                 (Status: 403) [Size: 290]                                 
/uploads              (Status: 301) [Size: 312] [--> http://10.10.10.68/uploads/]
/php                  (Status: 301) [Size: 308] [--> http://10.10.10.68/php/]    
/dev                  (Status: 301) [Size: 308] [--> http://10.10.10.68/dev/]    
/.                    (Status: 200) [Size: 7743]                                 
/fonts                (Status: 301) [Size: 310] [--> http://10.10.10.68/fonts/]  
/.htaccess            (Status: 403) [Size: 295]                                  
/.php3                (Status: 403) [Size: 291]                                  
/.phtml               (Status: 403) [Size: 292]                                  
/.htc                 (Status: 403) [Size: 290]                                  
/.php5                (Status: 403) [Size: 291]                                  
/.html_var_DE         (Status: 403) [Size: 298]                                  
/.php4                (Status: 403) [Size: 291]                                  
/server-status        (Status: 403) [Size: 299]                                  
/.htpasswd            (Status: 403) [Size: 295]                                  
/.html.               (Status: 403) [Size: 292]                                  
/.html.html           (Status: 403) [Size: 296]                                  
/.htpasswds           (Status: 403) [Size: 296]                                  
/.htm.                (Status: 403) [Size: 291]                                  
/.htmll               (Status: 403) [Size: 292]                                  
/.phps                (Status: 403) [Size: 291]                                  
/.html.old            (Status: 403) [Size: 295]                                  
/.ht                  (Status: 403) [Size: 289]                                  
/.html.bak            (Status: 403) [Size: 295]                                  
/.htm.htm             (Status: 403) [Size: 294]                                  
/.hta                 (Status: 403) [Size: 290]                                  
/.htgroup             (Status: 403) [Size: 294]                                  
/.html1               (Status: 403) [Size: 292]                                  
/.html.LCK            (Status: 403) [Size: 295]                                  
/.html.printable      (Status: 403) [Size: 301]                                  
/.htm.LCK             (Status: 403) [Size: 294]                                  
/.html.php            (Status: 403) [Size: 295]                                  
/.htaccess.bak        (Status: 403) [Size: 299]                                  
/.htx                 (Status: 403) [Size: 290]                                  
/.htmls               (Status: 403) [Size: 292]                                  
/.htlm                (Status: 403) [Size: 291]                                  
/.htm2                (Status: 403) [Size: 291]                                  
/.html-               (Status: 403) [Size: 292]                                  
/.htuser              (Status: 403) [Size: 293]                                  
                                                                                 
===============================================================
2021/05/05 09:06:50 Finished
===============================================================
```

This revealed a number of useful directories, including the `/dev` directory which would come in handy later.

# Tags

#writeup #oscp-prep #enum