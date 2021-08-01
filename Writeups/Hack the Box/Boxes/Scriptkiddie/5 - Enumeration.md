# Enumeration

## Autorecon

I ran autorecon against the box first:

```bash
autorecon 10.10.10.226
[*] Scanning target 10.10.10.226
[*] Running service detection nmap-top-20-udp on 10.10.10.226
[*] Running service detection nmap-full-tcp on 10.10.10.226
[*] Running service detection nmap-quick on 10.10.10.226
[*] Service detection nmap-quick on 10.10.10.226 finished successfully in 29 seconds
[*] Found ssh on tcp/22 on target 10.10.10.226
[*] Found http on tcp/5000 on target 10.10.10.226
[*] Running task tcp/22/sslscan on 10.10.10.226
[*] Running task tcp/22/nmap-ssh on 10.10.10.226
[*] Running task tcp/5000/sslscan on 10.10.10.226
[*] Running task tcp/5000/nmap-http on 10.10.10.226
[*] Running task tcp/5000/curl-index on 10.10.10.226
[*] Running task tcp/5000/curl-robots on 10.10.10.226
[*] Running task tcp/5000/wkhtmltoimage on 10.10.10.226
[*] Running task tcp/5000/whatweb on 10.10.10.226
[*] Task tcp/22/sslscan on 10.10.10.226 finished successfully in less than a second
[*] Task tcp/5000/sslscan on 10.10.10.226 finished successfully in less than a second
[*] Running task tcp/5000/nikto on 10.10.10.226
[*] Running task tcp/5000/gobuster on 10.10.10.226
[*] Task tcp/5000/curl-robots on 10.10.10.226 finished successfully in less than a second
[*] Task tcp/5000/curl-index on 10.10.10.226 finished successfully in less than a second
[*] Task tcp/22/nmap-ssh on 10.10.10.226 finished successfully in 7 seconds
[*] Task tcp/5000/wkhtmltoimage on 10.10.10.226 finished successfully in 11 seconds
[*] Task tcp/5000/whatweb on 10.10.10.226 finished successfully in 16 seconds
[*] [10:53:56] - There are 5 tasks still running on 10.10.10.226
[*] Service detection nmap-top-20-udp on 10.10.10.226 finished successfully in 1 minute, 43 seconds
[*] [10:54:56] - There are 4 tasks still running on 10.10.10.226
[*] [10:55:56] - There are 4 tasks still running on 10.10.10.226
[*] [10:56:56] - There are 4 tasks still running on 10.10.10.226
[*] [10:57:56] - There are 4 tasks still running on 10.10.10.226
[*] [10:58:56] - There are 4 tasks still running on 10.10.10.226
[*] [10:59:56] - There are 4 tasks still running on 10.10.10.226
[*] [11:00:56] - There are 4 tasks still running on 10.10.10.226
[*] [11:01:56] - There are 4 tasks still running on 10.10.10.226
[*] [11:02:56] - There are 4 tasks still running on 10.10.10.226
[*] [11:03:56] - There are 4 tasks still running on 10.10.10.226
[*] [11:04:56] - There are 4 tasks still running on 10.10.10.226
[*] [11:05:56] - There are 4 tasks still running on 10.10.10.226
[*] [11:06:56] - There are 4 tasks still running on 10.10.10.226
[*] [11:07:56] - There are 4 tasks still running on 10.10.10.226
[*] Task tcp/5000/nmap-http on 10.10.10.226 finished successfully in 14 minutes, 57 seconds
[*] [11:08:56] - There are 3 tasks still running on 10.10.10.226
[*] Task tcp/5000/nikto on 10.10.10.226 finished successfully in 16 minutes, 7 seconds
[*] [11:09:56] - There are 2 tasks still running on 10.10.10.226
[*] [11:10:56] - There are 2 tasks still running on 10.10.10.226
[*] [11:11:56] - There are 2 tasks still running on 10.10.10.226
[*] [11:12:56] - There are 2 tasks still running on 10.10.10.226
[*] Task tcp/5000/gobuster on 10.10.10.226 finished successfully in 19 minutes, 45 seconds
[*] [11:13:56] - There is 1 task still running on 10.10.10.226
...[snip]...
[*] [11:46:58] - There is 1 task still running on 10.10.10.226
```

I eventually cancelled the scan. I'm not sure what the task that never finished was (autorecon isn't great at telling you what exactly is running when something goes wrong) but it didn't turn out to matter.

## Nmap

The output of Autorecon's `_quick_tcp_nmap` scan:

```bash
# Nmap 7.91 scan initiated Sat Feb 13 10:52:56 2021 as: nmap -vv --reason -Pn -sV -sC --version-all -oN /root/Documents/scriptkiddie/results/10.10.10.226/scans/_quick_tcp_nmap.txt -oX /root/Documents/scriptkiddie/results/10.10.10.226/scans/xml/_quick_tcp_nmap.xml 10.10.10.226
Nmap scan report for 10.10.10.226
Host is up, received user-set (0.060s latency).
Scanned at 2021-02-13 10:52:59 GMT for 26s
Not shown: 998 closed ports
Reason: 998 resets
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC/YB1g/YHwZNvTzj8lysM+SzX6dZzRbfF24y3ywkhai4pViGEwUklIPkEvuLSGH97NJ4y8r9uUXzyoq3iuVJ/vGXiFlPCrg+QDp7UnwANBmDqbVLucKdor+JkWHJJ1h3ftpEHgol54tj+6J7ftmaOR29Iwg+FKtcyNG6PY434cfA0Pwshw6kKgFa+HWljNl+41H3WVua4QItPmrh+CrSoaA5kCe0FAP3c2uHcv2JyDjgCQxmN1GoLtlAsEznHlHI1wycNZGcHDnqxEmovPTN4qisOKEbYfy2mu1Eqq3Phv8UfybV8c60wUqGtClj3YOO1apDZKEe8eZZqy5eXU8mIO+uXcp5zxJ/Wrgng7WTguXGzQJiBHSFq52fHFvIYSuJOYEusLWkGhiyvITYLWZgnNL+qAVxZtP80ZTq+lm4cJHJZKl0OYsmqO0LjlMOMTPFyA+W2IOgAmnM+miSmSZ6n6pnSA+LE2Pj01egIhHw5+duAYxUHYOnKLVak1WWk/C68=
|   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJA31QhiIbYQMUwn/n3+qcrLiiJpYIia8HdgtwkI8JkCDm2n+j6dB3u5I17IOPXE7n5iPiW9tPF3Nb0aXmVJmlo=
|   256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOWjCdxetuUPIPnEGrowvR7qRAR7nuhUbfFraZFmbIr4
5000/tcp open  http    syn-ack ttl 63 Werkzeug httpd 0.16.1 (Python 3.8.5)
| http-methods: 
|_  Supported Methods: POST GET HEAD OPTIONS
|_http-server-header: Werkzeug/0.16.1 Python/3.8.5
|_http-title: k1d'5 h4ck3r t00l5
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Feb 13 10:53:25 2021 -- 1 IP address (1 host up) scanned in 28.98 seconds
```

Key findings:
- SSH on port 22
- Server running Ubuntu according to OpenSSL string
- Werkzeug webserver runing on port 5000. Running searchsploit against the version number didn't bring up anything that looked interesting

## Gobuster

I ran a quick gobuster against the domain:

```bash
┌──(mac㉿kali)-[~]
└─$ gobuster dir -u http://10.10.10.226:5000 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.226:5000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/12 12:46:02 Starting gobuster in directory enumeration mode
===============================================================
                                
===============================================================
2021/06/12 12:50:37 Finished
===============================================================
```

It didn't find anything.