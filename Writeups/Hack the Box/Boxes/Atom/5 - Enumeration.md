# Enumeration

## autorecon

I fired off `autorecon` first:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ autorecon 10.10.10.237
[*] Scanning target 10.10.10.237
[*] Running service detection nmap-full-tcp on 10.10.10.237
[*] Running service detection nmap-top-20-udp on 10.10.10.237
[*] Running service detection nmap-quick on 10.10.10.237
[!] Service detection nmap-top-20-udp on 10.10.10.237 returned non-zero exit code: 1
[*] [08:37:04] - There are 2 tasks still running on 10.10.10.237
[*] Service detection nmap-quick on 10.10.10.237 finished successfully in 1 minute, 1 second
[*] Found http on tcp/80 on target 10.10.10.237
[*] Found msrpc on tcp/135 on target 10.10.10.237
[*] Found ssl/http on tcp/443 on target 10.10.10.237
[*] Found microsoft-ds on tcp/445 on target 10.10.10.237
[!] [tcp/445/nbtscan] Scan cannot be run against tcp port 445. Skipping.
[*] Running task tcp/80/sslscan on 10.10.10.237
[*] Running task tcp/80/nmap-http on 10.10.10.237
[*] Running task tcp/80/curl-index on 10.10.10.237
[*] Running task tcp/80/curl-robots on 10.10.10.237
[*] Running task tcp/80/wkhtmltoimage on 10.10.10.237
[*] Running task tcp/80/whatweb on 10.10.10.237
[*] Running task tcp/80/nikto on 10.10.10.237
[*] Running task tcp/80/gobuster on 10.10.10.237
[*] Running task tcp/135/sslscan on 10.10.10.237
[*] Task tcp/80/sslscan on 10.10.10.237 finished successfully in less than a second
[*] Running task tcp/135/nmap-msrpc on 10.10.10.237
[*] Task tcp/135/sslscan on 10.10.10.237 finished successfully in less than a second
[*] Running task tcp/443/sslscan on 10.10.10.237
[*] Task tcp/80/curl-robots on 10.10.10.237 finished successfully in 1 second
[*] Task tcp/80/curl-index on 10.10.10.237 finished successfully in 1 second
[*] Running task tcp/443/nmap-http on 10.10.10.237
[*] Running task tcp/443/curl-index on 10.10.10.237
[*] Task tcp/443/curl-index on 10.10.10.237 finished successfully in less than a second
[*] Running task tcp/443/curl-robots on 10.10.10.237
[!] Task tcp/80/gobuster on 10.10.10.237 returned non-zero exit code: 1
[*] Running task tcp/443/wkhtmltoimage on 10.10.10.237
[*] Task tcp/443/curl-robots on 10.10.10.237 finished successfully in less than a second
[*] Running task tcp/443/whatweb on 10.10.10.237
[*] Task tcp/80/wkhtmltoimage on 10.10.10.237 finished successfully in 13 seconds
[*] Running task tcp/443/nikto on 10.10.10.237
[*] Task tcp/443/wkhtmltoimage on 10.10.10.237 finished successfully in 11 seconds
[*] Running task tcp/443/gobuster on 10.10.10.237
[!] Task tcp/443/gobuster on 10.10.10.237 returned non-zero exit code: 1
[*] Running task tcp/445/sslscan on 10.10.10.237
[*] Task tcp/445/sslscan on 10.10.10.237 finished successfully in less than a second
[*] Running task tcp/445/nmap-smb on 10.10.10.237
[*] Task tcp/443/sslscan on 10.10.10.237 finished successfully in 13 seconds
[*] Running task tcp/445/enum4linux on 10.10.10.237
[*] Task tcp/135/nmap-msrpc on 10.10.10.237 finished successfully in 24 seconds
[*] Running task tcp/445/smbclient on 10.10.10.237
[*] Task tcp/445/smbclient on 10.10.10.237 finished successfully in 1 second
[*] Running task tcp/445/smbmap-share-permissions on 10.10.10.237
[*] Task tcp/80/whatweb on 10.10.10.237 finished successfully in 28 seconds
[*] Running task tcp/445/smbmap-list-contents on 10.10.10.237
[*] Task tcp/80/nmap-http on 10.10.10.237 finished successfully in 30 seconds
[*] Running task tcp/445/smbmap-execute-command on 10.10.10.237
[*] Task tcp/443/whatweb on 10.10.10.237 finished successfully in 30 seconds
[*] Task tcp/445/enum4linux on 10.10.10.237 finished successfully in 23 seconds
[*] Task tcp/445/smbmap-execute-command on 10.10.10.237 finished successfully in 12 seconds
[*] Task tcp/445/smbmap-share-permissions on 10.10.10.237 finished successfully in 17 seconds
[*] Task tcp/445/smbmap-list-contents on 10.10.10.237 finished successfully in 14 seconds
[*] Task tcp/443/nmap-http on 10.10.10.237 finished successfully in 51 seconds
[*] [08:38:04] - There are 4 tasks still running on 10.10.10.237
[*] Task tcp/445/nmap-smb on 10.10.10.237 finished successfully in 1 minute, 45 seconds
[*] [08:39:04] - There are 3 tasks still running on 10.10.10.237
[*] [08:40:04] - There are 3 tasks still running on 10.10.10.237
[*] Service detection nmap-full-tcp on 10.10.10.237 finished successfully in 4 minutes, 48 seconds
[*] Found http on tcp/5985 on target 10.10.10.237
[*] Found redis on tcp/6379 on target 10.10.10.237
[*] Found pando-pub on tcp/7680 on target 10.10.10.237
[*] Running task tcp/5985/sslscan on 10.10.10.237
[*] Running task tcp/5985/nmap-http on 10.10.10.237
[*] Running task tcp/5985/curl-index on 10.10.10.237
[*] Running task tcp/5985/curl-robots on 10.10.10.237
[*] Running task tcp/5985/wkhtmltoimage on 10.10.10.237
[*] Running task tcp/5985/whatweb on 10.10.10.237
[*] Running task tcp/5985/nikto on 10.10.10.237
[*] Running task tcp/5985/gobuster on 10.10.10.237
[*] Task tcp/5985/sslscan on 10.10.10.237 finished successfully in less than a second
[*] Running task tcp/6379/sslscan on 10.10.10.237
[*] Task tcp/6379/sslscan on 10.10.10.237 finished successfully in less than a second
[*] Running task tcp/7680/sslscan on 10.10.10.237
[*] Task tcp/7680/sslscan on 10.10.10.237 finished successfully in less than a second
[*] Task tcp/5985/curl-robots on 10.10.10.237 finished successfully in 1 second
[*] Task tcp/5985/curl-index on 10.10.10.237 finished successfully in 1 second
[!] Task tcp/5985/gobuster on 10.10.10.237 returned non-zero exit code: 1
[!] Task tcp/5985/wkhtmltoimage on 10.10.10.237 returned non-zero exit code: 1
[*] Task tcp/5985/whatweb on 10.10.10.237 finished successfully in 9 seconds
[*] [08:41:04] - There are 4 tasks still running on 10.10.10.237
[*] [08:42:04] - There are 4 tasks still running on 10.10.10.237
[*] Task tcp/80/nikto on 10.10.10.237 finished successfully in 5 minutes, 34 seconds
[*] [08:43:04] - There are 3 tasks still running on 10.10.10.237
[*] Task tcp/5985/nmap-http on 10.10.10.237 finished successfully in 2 minutes, 19 seconds
[*] [08:44:04] - There are 2 tasks still running on 10.10.10.237
[*] [08:45:04] - There are 2 tasks still running on 10.10.10.237
[*] [08:46:04] - There are 2 tasks still running on 10.10.10.237
[*] [08:47:04] - There are 2 tasks still running on 10.10.10.237
[*] Task tcp/5985/nikto on 10.10.10.237 finished successfully in 7 minutes, 6 seconds
[*] [08:48:04] - There is 1 task still running on 10.10.10.237
[*] [08:49:04] - There is 1 task still running on 10.10.10.237
[*] [08:50:05] - There is 1 task still running on 10.10.10.237
[*] [08:51:05] - There is 1 task still running on 10.10.10.237
[*] [08:52:05] - There is 1 task still running on 10.10.10.237
[*] [08:53:05] - There is 1 task still running on 10.10.10.237
[*] Task tcp/443/nikto on 10.10.10.237 finished successfully in 16 minutes, 35 seconds
[*] Finished scanning target 10.10.10.237 in 17 minutes, 49 seconds
[*] Finished scanning all targets in 17 minutes, 49 seconds!

```

## nmap

I then took a look at the nmap results.

### Key Results

Open Ports:
- 80 (HTTP) - Running Apache
- 135 (Windows RPC)
- 443 (HTTPS) - Certificate does not expose Domain Name
- 445 (SMB)
- 5985 (WinRM)
- 6379 (Redis)
- 7680 (Windows Update Delivery Optimisation)

Service Info: Host: ATOM; OS: Windows; CPE: cpe:/o:microsoft:windows

### Full Port Scan

```bash
┌──(mac㉿kali)-[~Documents/HTB/atom/results/10.10.10.237/scans]
└─$ cat _full_tcp_nmap.txt 
# Nmap 7.91 scan initiated Mon Apr 19 08:36:05 2021 as: nmap -vv --reason -Pn -A --osscan-guess --version-all -p- -oN /home/mac/results/10.10.10.237/scans/_full_tcp_nmap.txt -oX /home/mac/results/10.10.10.237/scans/xml/_full_tcp_nmap.xml 10.10.10.237
Nmap scan report for 10.10.10.237
Host is up, received user-set (0.029s latency).
Scanned at 2021-04-19 08:36:07 BST for 284s
Not shown: 65528 filtered ports
Reason: 65528 no-responses
PORT     STATE SERVICE      REASON  VERSION
80/tcp   open  http         syn-ack Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
| http-methods: 
|   Supported Methods: GET POST OPTIONS HEAD TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: Heed Solutions
135/tcp  open  msrpc        syn-ack Microsoft Windows RPC
443/tcp  open  ssl/http     syn-ack Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
| http-methods: 
|   Supported Methods: GET POST OPTIONS HEAD TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: Heed Solutions
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2009-11-10T23:48:47
| Not valid after:  2019-11-08T23:48:47
| MD5:   a0a4 4cc9 9e84 b26f 9e63 9f9e d229 dee0
| SHA-1: b023 8c54 7a90 5bfa 119c 4e8b acca eacf 3649 1ff6
| -----BEGIN CERTIFICATE-----
| MIIBnzCCAQgCCQC1x1LJh4G1AzANBgkqhkiG9w0BAQUFADAUMRIwEAYDVQQDEwls
| b2NhbGhvc3QwHhcNMDkxMTEwMjM0ODQ3WhcNMTkxMTA4MjM0ODQ3WjAUMRIwEAYD
| VQQDEwlsb2NhbGhvc3QwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMEl0yfj
| 7K0Ng2pt51+adRAj4pCdoGOVjx1BmljVnGOMW3OGkHnMw9ajibh1vB6UfHxu463o
| J1wLxgxq+Q8y/rPEehAjBCspKNSq+bMvZhD4p8HNYMRrKFfjZzv3ns1IItw46kgT
| gDpAl1cMRzVGPXFimu5TnWMOZ3ooyaQ0/xntAgMBAAEwDQYJKoZIhvcNAQEFBQAD
| gYEAavHzSWz5umhfb/MnBMa5DL2VNzS+9whmmpsDGEG+uR0kM1W2GQIdVHHJTyFd
| aHXzgVJBQcWTwhp84nvHSiQTDBSaT6cQNQpvag/TaED/SEQpm0VqDFwpfFYuufBL
| vVNbLkKxbK2XwUvu0RxoLdBMC/89HqrZ0ppiONuQ+X2MtxE=
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds syn-ack Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)
5985/tcp open  http         syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
6379/tcp open  redis        syn-ack Redis key-value store
7680/tcp open  pando-pub?   syn-ack
Service Info: Host: ATOM; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h30m58s, deviation: 4h02m30s, median: 10m57s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 58850/tcp): CLEAN (Timeout)
|   Check 2 (port 38781/tcp): CLEAN (Timeout)
|   Check 3 (port 39922/udp): CLEAN (Timeout)
|   Check 4 (port 16707/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 10 Pro 19042 (Windows 10 Pro 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: ATOM
|   NetBIOS computer name: ATOM\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-04-19T00:51:10-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-04-19T07:51:11
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Apr 19 08:40:51 2021 -- 1 IP address (1 host up) scanned in 286.66 seconds
```

## SMB Enumeration

Autorecon ran the following commands:

```bash
smbclient -L\\ -N -I 10.10.10.237 2>&1 | tee "/home/mac/results/10.10.10.237/scans/smbclient.txt"

smbmap -H 10.10.10.237 -P 445 2>&1 | tee -a "/home/mac/results/10.10.10.237/scans/smbmap-share-permissions.txt"; smbmap -u null -p "" -H 10.10.10.237 -P 445 2>&1 | tee -a "/home/mac/results/10.10.10.237/scans/smbmap-share-permissions.txt"

smbmap -H 10.10.10.237 -P 445 -R 2>&1 | tee -a "/home/mac/results/10.10.10.237/scans/smbmap-list-contents.txt"; smbmap -u null -p "" -H 10.10.10.237 -P 445 -R 2>&1 | tee -a "/home/mac/results/10.10.10.237/scans/smbmap-list-contents.txt"

smbmap -H 10.10.10.237 -P 445 -x "ipconfig /all" 2>&1 | tee -a "/home/mac/results/10.10.10.237/scans/smbmap-execute-command.txt"; smbmap -u null -p "" -H 10.10.10.237 -P 445 -x "ipconfig /all" 2>&1 | tee -a "/home/mac/results/10.10.10.237/scans/smbmap-execute-command.txt"
```

I'd not used much `smbmap` myself, so I ran some of the commands manually to familiarise myself with how they worked.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom/results]
└─$ smbmap -H 10.10.10.237 -P 445
[!] Authentication error on 10.10.10.237
┌──(mac㉿kali)-[~/Documents/HTB/atom/results]
└─$ smbmap -u null -p "" -H 10.10.10.237 -P 445
[+] Guest session   	IP: 10.10.10.237:445	Name: 10.10.10.237                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	Software_Updates                                  	READ, WRITE	
```

Null authentication lets us list the shares on the box.

Autorecon also ran the following `nmap` scan on port 445 to enumerate shares:

```
# Nmap 7.91 scan initiated Mon Apr 19 08:37:19 2021 as: nmap -vv --reason -Pn -sV -p 445 "--script=banner,(nbstat or smb* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" --script-args=unsafe=1 -oN /home/mac/results/10.10.10.237/scans/tcp_445_smb_nmap.txt -oX /home/mac/results/10.10.10.237/scans/xml/tcp_445_smb_nmap.xml 10.10.10.237
Nmap scan report for 10.10.10.237
Host is up, received user-set (0.024s latency).
Scanned at 2021-04-19 08:37:24 BST for 99s

PORT    STATE SERVICE      REASON  VERSION
445/tcp open  microsoft-ds syn-ack Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)
Service Info: Host: ATOM; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.10.237\ADMIN$: 
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Remote Admin
|     Anonymous access: <none>
|     Current user access: <none>
|   \\10.10.10.237\C$: 
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Default share
|     Anonymous access: <none>
|     Current user access: <none>
|   \\10.10.10.237\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: Remote IPC
|     Anonymous access: <none>
|     Current user access: READ/WRITE
|   \\10.10.10.237\Software_Updates: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Anonymous access: <none>
|_    Current user access: READ/WRITE
| smb-ls: Volume \\10.10.10.237\Software_Updates
| SIZE   TIME                 FILENAME
| <DIR>  2021-03-30T20:43:48  .
| <DIR>  2021-03-30T20:43:48  ..
| <DIR>  2021-04-19T07:47:04  client1
| <DIR>  2021-04-19T07:47:04  client2
| <DIR>  2021-04-19T07:47:04  client3
| 35202  2021-04-13T09:36:28  UAT_Testing_Procedures.pdf
|_
| smb-mbenum: 
|_  ERROR: Call to Browser Service failed with status = 2184
| smb-os-discovery: 
|   OS: Windows 10 Pro 19042 (Windows 10 Pro 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: ATOM
|   NetBIOS computer name: ATOM\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-04-19T00:48:53-07:00
|_smb-print-text: false
| smb-protocols: 
|   dialects: 
|     NT LM 0.12 (SMBv1) [dangerous, but default]
|     2.02
|     2.10
|     3.00
|     3.02
|_    3.11
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
| smb2-capabilities: 
|   2.02: 
|     Distributed File System
|   2.10: 
|     Distributed File System
|     Leasing
|     Multi-credit operations
|   3.00: 
|     Distributed File System
|     Leasing
|     Multi-credit operations
|   3.02: 
|     Distributed File System
|     Leasing
|     Multi-credit operations
|   3.11: 
|     Distributed File System
|     Leasing
|_    Multi-credit operations
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-04-19T07:48:51
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Apr 19 08:39:03 2021 -- 1 IP address (1 host up) scanned in 104.11 seconds
```

This shows that two shares have read access, and that there are some files that we can get from the server, including a PDF.

## gobuster

I ran a simple directory bust with `gobuster`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ gobuster dir -u http://10.10.10.237 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.237
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/04/19 14:20:36 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 338] [--> http://10.10.10.237/images/]
/.html                (Status: 403) [Size: 302]                                  
/.htm                 (Status: 403) [Size: 302]                                  
/webalizer            (Status: 403) [Size: 302]                                  
/Images               (Status: 301) [Size: 338] [--> http://10.10.10.237/Images/]
/.                    (Status: 200) [Size: 7581]                                 
/phpmyadmin           (Status: 403) [Size: 302]                                  
/.htaccess            (Status: 403) [Size: 302]                                  
/examples             (Status: 503) [Size: 402]                                  
/.htc                 (Status: 403) [Size: 302]                                  
/releases             (Status: 301) [Size: 340] [--> http://10.10.10.237/releases/]
/IMAGES               (Status: 301) [Size: 338] [--> http://10.10.10.237/IMAGES/]  
/.html_var_DE         (Status: 403) [Size: 302]                                    
/licenses             (Status: 403) [Size: 421]                                    
/server-status        (Status: 403) [Size: 421]                                    
/.htpasswd            (Status: 403) [Size: 302]                                    
/con                  (Status: 403) [Size: 302]                                    
/.html.               (Status: 403) [Size: 302]                                    
/.html.html           (Status: 403) [Size: 302]                                    
/.htpasswds           (Status: 403) [Size: 302]                                    
/.htm.                (Status: 403) [Size: 302]                                    
/.htmll               (Status: 403) [Size: 302]                                    
/.html.old            (Status: 403) [Size: 302]                                    
/Releases             (Status: 301) [Size: 340] [--> http://10.10.10.237/Releases/]
/.ht                  (Status: 403) [Size: 302]                                    
/.html.bak            (Status: 403) [Size: 302]                                    
/.htm.htm             (Status: 403) [Size: 302]                                    
/aux                  (Status: 403) [Size: 302]                                    
/.hta                 (Status: 403) [Size: 302]                                    
/.htgroup             (Status: 403) [Size: 302]                                    
/.html1               (Status: 403) [Size: 302]                                    
/.html.printable      (Status: 403) [Size: 302]                                    
/.html.LCK            (Status: 403) [Size: 302]                                    
/prn                  (Status: 403) [Size: 302]                                    
/.htm.LCK             (Status: 403) [Size: 302]                                    
/.htaccess.bak        (Status: 403) [Size: 302]                                    
/.html.php            (Status: 403) [Size: 302]                                    
/.htmls               (Status: 403) [Size: 302]                                    
/.htx                 (Status: 403) [Size: 302]                                    
/server-info          (Status: 403) [Size: 421]                                    
/.htlm                (Status: 403) [Size: 302]                                    
/.htm2                (Status: 403) [Size: 302]                                    
/.html-               (Status: 403) [Size: 302]                                    
/.htuser              (Status: 403) [Size: 302]                                    
                                                                                   
===============================================================
2021/04/19 14:22:37 Finished
===============================================================
```

It didn't find anything interesting.

### Vhosts

I also ran a virtual host scan to see if there were any valid subdomains:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ gobuster vhost -u atom.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://atom.htb
[+] Method:       GET
[+] Threads:      10
[+] Wordlist:     /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2021/04/20 17:09:53 Starting gobuster in VHOST enumeration mode
===============================================================
                                  
===============================================================
2021/04/20 17:19:26 Finished
===============================================================
```

Nothing here either.