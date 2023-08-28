# Enumeration

## Autorecon

I started off with an `autorecon` scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ autorecon 10.10.10.233
[*] Scanning target 10.10.10.233
[*] Running service detection nmap-full-tcp on 10.10.10.233
[*] Running service detection nmap-top-20-udp on 10.10.10.233
[*] Running service detection nmap-quick on 10.10.10.233
[!] Service detection nmap-top-20-udp on 10.10.10.233 returned non-zero exit code: 1
[*] Service detection nmap-quick on 10.10.10.233 finished successfully in 42 seconds
[*] Found ssh on tcp/22 on target 10.10.10.233
[*] Found http on tcp/80 on target 10.10.10.233
[*] Running task tcp/22/sslscan on 10.10.10.233
[*] Running task tcp/22/nmap-ssh on 10.10.10.233
[*] Running task tcp/80/sslscan on 10.10.10.233
[*] Running task tcp/80/nmap-http on 10.10.10.233
[*] Running task tcp/80/curl-index on 10.10.10.233
[*] Running task tcp/80/curl-robots on 10.10.10.233
[*] Running task tcp/80/wkhtmltoimage on 10.10.10.233
[*] Running task tcp/80/whatweb on 10.10.10.233
[*] Running task tcp/80/nikto on 10.10.10.233
[*] Task tcp/22/sslscan on 10.10.10.233 finished successfully in less than a second
[*] Task tcp/80/sslscan on 10.10.10.233 finished successfully in less than a second
[*] Running task tcp/80/gobuster on 10.10.10.233
[*] Task tcp/80/curl-robots on 10.10.10.233 finished successfully in 1 second
[*] Task tcp/80/curl-index on 10.10.10.233 finished successfully in 2 seconds
[!] Task tcp/80/gobuster on 10.10.10.233 returned non-zero exit code: 1
[*] [10:12:57] - There are 6 tasks still running on 10.10.10.233
[*] Task tcp/22/nmap-ssh on 10.10.10.233 finished successfully in 19 seconds
[*] Task tcp/80/wkhtmltoimage on 10.10.10.233 finished successfully in 30 seconds
[*] Task tcp/80/whatweb on 10.10.10.233 finished successfully in 51 seconds
[*] [10:13:58] - There are 3 tasks still running on 10.10.10.233
[*] Task tcp/80/nmap-http on 10.10.10.233 finished successfully in 1 minute, 26 seconds
[*] [10:14:58] - There are 2 tasks still running on 10.10.10.233
[*] [10:15:58] - There are 2 tasks still running on 10.10.10.233
[*] [10:16:58] - There are 2 tasks still running on 10.10.10.233
[*] [10:17:58] - There are 2 tasks still running on 10.10.10.233
[*] [10:18:58] - There are 2 tasks still running on 10.10.10.233
[*] [10:19:58] - There are 2 tasks still running on 10.10.10.233
[*] [10:20:58] - There are 2 tasks still running on 10.10.10.233
[*] [10:21:58] - There are 2 tasks still running on 10.10.10.233
[*] [10:22:58] - There are 2 tasks still running on 10.10.10.233
[*] [10:23:58] - There are 2 tasks still running on 10.10.10.233
[*] Service detection nmap-full-tcp on 10.10.10.233 finished successfully in 12 minutes, 25 seconds
[*] [10:24:58] - There is 1 task still running on 10.10.10.233
[*] [10:25:58] - There is 1 task still running on 10.10.10.233
[*] [10:26:58] - There is 1 task still running on 10.10.10.233
[*] [10:27:58] - There is 1 task still running on 10.10.10.233
[*] [10:28:58] - There is 1 task still running on 10.10.10.233
[*] [10:29:58] - There is 1 task still running on 10.10.10.233
[*] [10:30:58] - There is 1 task still running on 10.10.10.233
[*] [10:31:58] - There is 1 task still running on 10.10.10.233
[*] [10:32:58] - There is 1 task still running on 10.10.10.233
[*] [10:33:58] - There is 1 task still running on 10.10.10.233
[*] Task tcp/80/nikto on 10.10.10.233 finished successfully in 21 minutes, 50 seconds
[*] Finished scanning target 10.10.10.233 in 22 minutes, 32 seconds
[*] Finished scanning all targets in 22 minutes, 38 seconds!
```

A couple of the scans failed (annoyingly, one of them was Gobuster) but we can re-run them manually.

It pretty quickly found a website and SSH. Let's review the results from the `nmap` scan.

## Nmap

The UDP scan failed, but the regular scan worked.

I picked up a tip [on reddit](https://www.reddit.com/r/oscp/comments/k7x4o1/just_passed_oscpmy_journey_and_tips/) to start a web server in the autorecon `results` directory, to be able to easily view the scan outputs in a browser. To do so, run the following commands:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ cd results/10.10.10.233/
┌──(mac㉿kali)-[~/Documents/HTB/armageddon/results/10.10.10.233]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

You can then visit `http://localhost:8000/scans/_quick_tcp_nmap.txt` to see the quick nmap scan:

```
# Nmap 7.91 scan initiated Tue Mar 30 10:11:58 2021 as: nmap -vv --reason -Pn -sV -sC --version-all -oN /home/mac/Documents/HTB/armageddon/results/10.10.10.233/scans/_quick_tcp_nmap.txt -oX /home/mac/Documents/HTB/armageddon/results/10.10.10.233/scans/xml/_quick_tcp_nmap.xml 10.10.10.233
Nmap scan report for 10.10.10.233
Host is up, received user-set (0.13s latency).
Scanned at 2021-03-30 10:12:03 BST for 35s
Not shown: 998 closed ports
Reason: 998 conn-refused
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 82:c6:bb:c7:02:6a:93:bb:7c:cb:dd:9c:30:93:79:34 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDC2xdFP3J4cpINVArODYtbhv+uQNECQHDkzTeWL+4aLgKcJuIoA8dQdVuP2UaLUJ0XtbyuabPEBzJl3IHg3vztFZ8UEcS94KuWP09ghv6fhc7JbFYONVJTYLiEPD8nrS/V2EPEQJ2ubNXcZAR76X9SZqt11JTyQH/s6tPH+m3m/84NUU8PNb/dyhrFpCUmZzzJQ1zCDStLXJnCAOE7EfW2wNm1CBPCXn1wNvO3SKwokCm4GoMKHSM9rNb9FjGLIY0nq+8mt7RTJZ+WLdHsje3AkBk1yooGFF+0TdOj42YK2OtAKDQBWnBm1nqLQsmm/Va9T2bPYLLK5aUd4/578u7h
|   256 3a:ca:95:30:f3:12:d7:ca:45:05:bc:c7:f1:16:bb:fc (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBE4kP4gQ5Th3eu3vz/kPWwlUCm+6BSM6M3Y43IuYVo3ppmJG+wKiabo/gVYLOwzG7js497Vr7eGIgsjUtbIGUrY=
|   256 7a:d4:b3:68:79:cf:62:8a:7d:5a:61:e7:06:0f:5f:33 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG9ZlC3EA13xZbzvvdjZRWhnu9clFOUe7irG8kT0oR4A
80/tcp open  http    syn-ack Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-favicon: Unknown favicon MD5: 1487A9908F898326EBABFFFD2407920D
|_http-generator: Drupal 7 (http://drupal.org)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 36 disallowed entries 
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
| /LICENSE.txt /MAINTAINERS.txt /update.php /UPGRADE.txt /xmlrpc.php 
| /admin/ /comment/reply/ /filter/tips/ /node/add/ /search/ 
| /user/register/ /user/password/ /user/login/ /user/logout/ /?q=admin/ 
| /?q=comment/reply/ /?q=filter/tips/ /?q=node/add/ /?q=search/ 
|_/?q=user/password/ /?q=user/register/ /?q=user/login/ /?q=user/logout/
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Welcome to  Armageddon |  Armageddon

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Mar 30 10:12:38 2021 -- 1 IP address (1 host up) scanned in 40.68 seconds
```

This shows we have port 22 and 80 open, suggesting SSH and a webserver.

It's good practice to come back to the full nmap scan (`http://localhost:8000/scans/_full_tcp_nmap.txt`) once it's finished, although it usually takes a little longer. In this case, it didn't find anything that wasn't found in the quick scan.

The scan reveals a number of interesting entries in `/robots.txt`, including `xmlrpc.php` which stands out immediately - this is a vulnerable feature of some Wordpress blogs.

It also reveals Drupal to be running on the server, which is a popular CMS.

## Gobuster

Gobuster failed to run during autorecon, so we can re-run it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon/results/10.10.10.233]
└─$ gobuster dir -u http://10.10.10.233 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.233
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/03/30 10:45:29 Starting gobuster in directory enumeration mode
===============================================================
/modules              (Status: 301) [Size: 236] [--> http://10.10.10.233/modules/]
/.html                (Status: 403) [Size: 207]                                   
/includes             (Status: 301) [Size: 237] [--> http://10.10.10.233/includes/]
/themes               (Status: 301) [Size: 235] [--> http://10.10.10.233/themes/]  
/scripts              (Status: 301) [Size: 236] [--> http://10.10.10.233/scripts/] 
/misc                 (Status: 301) [Size: 233] [--> http://10.10.10.233/misc/]    
/.htm                 (Status: 403) [Size: 206]                                    
/profiles             (Status: 301) [Size: 237] [--> http://10.10.10.233/profiles/]
/sites                (Status: 301) [Size: 234] [--> http://10.10.10.233/sites/]   
/.htaccess            (Status: 403) [Size: 211]                                    
/.htc                 (Status: 403) [Size: 206]                                    
/.html_var_DE         (Status: 403) [Size: 214]                                    
/.htpasswd            (Status: 403) [Size: 211]                                    
/.html.               (Status: 403) [Size: 208]                                    
/.html.html           (Status: 403) [Size: 212]                                    
/.htpasswds           (Status: 403) [Size: 212]                                    
/.htm.                (Status: 403) [Size: 207]                                    
/.htmll               (Status: 403) [Size: 208]                                    
/.html.old            (Status: 403) [Size: 211]                                    
/.html.bak            (Status: 403) [Size: 211]                                    
/.ht                  (Status: 403) [Size: 205]                                    
/.htm.htm             (Status: 403) [Size: 210]                                    
/.gitignore           (Status: 200) [Size: 174]                                    
/.hta                 (Status: 403) [Size: 206]                                    
/.htgroup             (Status: 403) [Size: 210]                                    
/.html1               (Status: 403) [Size: 208]                                    
/.html.LCK            (Status: 403) [Size: 211]                                    
/.html.printable      (Status: 403) [Size: 217]                                    
/.htm.LCK             (Status: 403) [Size: 210]                                    
/.htaccess.bak        (Status: 403) [Size: 215]                                    
/.html.php            (Status: 403) [Size: 211]                                    
/.htx                 (Status: 403) [Size: 206]                                    
/.htmls               (Status: 403) [Size: 208]                                    
/.htm2                (Status: 403) [Size: 207]                                    
/.htlm                (Status: 403) [Size: 207]                                    
/.htuser              (Status: 403) [Size: 209]                                    
/.html-               (Status: 403) [Size: 208]                                    
                                                                                   
===============================================================
2021/03/30 10:53:15 Finished
===============================================================
```

It found a few Wordpress-related directories, but nothing that immediately stands out.