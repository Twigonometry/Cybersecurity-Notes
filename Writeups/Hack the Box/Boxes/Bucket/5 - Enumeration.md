# Enumeration

## nmap

I started with an `nmap` scan to discover open ports:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket]
└─$ nmap 10.10.10.212 -sC -sV -oA nmap/  
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-15 15:04 GMT  
Nmap scan report for 10.10.10.212  
Host is up (0.032s latency).  
Not shown: 998 closed ports  
PORT STATE SERVICE VERSION  
22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)  
80/tcp open http Apache httpd 2.4.41  
|_http-server-header: Apache/2.4.41 (Ubuntu)  
|_http-title: Did not follow redirect to http://bucket.htb/
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux\kernel  
  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 8.45 seconds
```

This shows only ports 22 and 80 are open, for SSH and HTTP. This means we should start by looking at the [[Writeups/Hack the Box/Boxes/Bucket/10 - Website|website]]

## Gobuster

I ran gobuster on the initial website domain:

```bash
┌──(mac㉿kali)-[~/Documents/enum]
└─$ gobuster dir -u http://10.10.10.212 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.212
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/04/28 12:31:01 Starting gobuster in directory enumeration mode
===============================================================
Error: the server returns a status code that matches the provided options for non existing urls. http://10.10.10.212/84530f45-4eb0-4f43-bae7-e0227949c00c => 302 (Length: 280). To continue please exclude the status code, the length or use the --wildcard switch
```

Running with the `--wildcard` switch returns a large number of `302` status codes.

When I discovered the `bucket.htb` domain, I re-ran the scan:

```bash
┌──(mac㉿kali)-[~/Documents/enum]
└─$ gobuster dir -u http://bucket.htb --wildcard -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://bucket.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/04/28 12:33:51 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/.htm                 (Status: 403) [Size: 275]
/.                    (Status: 200) [Size: 5344]
/.htaccess            (Status: 403) [Size: 275] 
/.phtml               (Status: 403) [Size: 275] 
/.htc                 (Status: 403) [Size: 275] 
/.html_var_DE         (Status: 403) [Size: 275] 
/server-status        (Status: 403) [Size: 275] 
/.htpasswd            (Status: 403) [Size: 275] 
/.html.               (Status: 403) [Size: 275] 
/.html.html           (Status: 403) [Size: 275] 
/.htpasswds           (Status: 403) [Size: 275] 
/.htm.                (Status: 403) [Size: 275] 
/.htmll               (Status: 403) [Size: 275] 
/.phps                (Status: 403) [Size: 275] 
/.html.old            (Status: 403) [Size: 275] 
/.ht                  (Status: 403) [Size: 275] 
/.html.bak            (Status: 403) [Size: 275] 
/.htm.htm             (Status: 403) [Size: 275] 
/.hta                 (Status: 403) [Size: 275] 
/.html1               (Status: 403) [Size: 275] 
/.htgroup             (Status: 403) [Size: 275] 
/.html.LCK            (Status: 403) [Size: 275] 
/.html.printable      (Status: 403) [Size: 275] 
/.htm.LCK             (Status: 403) [Size: 275] 
/.htaccess.bak        (Status: 403) [Size: 275] 
/.html.php            (Status: 403) [Size: 275] 
/.htmls               (Status: 403) [Size: 275] 
/.htx                 (Status: 403) [Size: 275] 
/.htlm                (Status: 403) [Size: 275] 
/.htm2                (Status: 403) [Size: 275] 
/.html-               (Status: 403) [Size: 275] 
/.htuser              (Status: 403) [Size: 275] 
                                                
===============================================================
2021/04/28 12:35:26 Finished
===============================================================
```

There were no useful results here.

### s3.bucket.htb

After discovering the `s3` subdomain, I ran gobuster against it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket]
└─$ gobuster dir -u s3.bucket.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://s3.bucket.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/04/29 13:52:34 Starting gobuster in directory enumeration mode
===============================================================
/health               (Status: 200) [Size: 54]
/shell                (Status: 200) [Size: 0] 
/server-status        (Status: 403) [Size: 278]
/shells               (Status: 500) [Size: 158]
                                               
===============================================================
2021/04/29 14:00:08 Finished
===============================================================
```

This revealed the `/health` and `shell` pages.