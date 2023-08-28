# Enumeration

## Autorecon

I started off with autorecon

```bash
┌──(mac㉿kali)-[~/.config/AutoRecon]
└─$ autorecon 10.10.10.138
[*] Scanning target 10.10.10.138
[*] Running service detection nmap-full-tcp on 10.10.10.138
[*] Running service detection nmap-top-20-udp on 10.10.10.138
[*] Running service detection nmap-quick on 10.10.10.138
[!] Service detection nmap-top-20-udp on 10.10.10.138 returned non-zero exit code: 1
[*] Service detection nmap-quick on 10.10.10.138 finished successfully in 25 seconds
[*] Found ssh on tcp/22 on target 10.10.10.138
[*] Found http on tcp/80 on target 10.10.10.138
[*] Running task tcp/22/sslscan on 10.10.10.138
[*] Running task tcp/22/nmap-ssh on 10.10.10.138
[*] Running task tcp/80/sslscan on 10.10.10.138
[*] Running task tcp/80/nmap-http on 10.10.10.138
[*] Running task tcp/80/curl-index on 10.10.10.138
[*] Running task tcp/80/curl-robots on 10.10.10.138
[*] Running task tcp/80/wkhtmltoimage on 10.10.10.138
[*] Running task tcp/80/whatweb on 10.10.10.138
[*] Running task tcp/80/nikto on 10.10.10.138
[*] Task tcp/22/sslscan on 10.10.10.138 finished successfully in 1 second
[*] Running task tcp/80/gobuster on 10.10.10.138
[*] Task tcp/80/sslscan on 10.10.10.138 finished successfully in 1 second
[!] Task tcp/80/gobuster on 10.10.10.138 returned non-zero exit code: 1
[*] Task tcp/80/curl-index on 10.10.10.138 finished successfully in 4 seconds
[*] Task tcp/80/curl-robots on 10.10.10.138 finished successfully in 7 seconds
[*] Task tcp/22/nmap-ssh on 10.10.10.138 finished successfully in 14 seconds
[*] Task tcp/80/nikto on 10.10.10.138 finished successfully in 16 seconds
[*] Task tcp/80/nmap-http on 10.10.10.138 finished successfully in 16 seconds
[!] Task tcp/80/wkhtmltoimage on 10.10.10.138 returned non-zero exit code: 1
[*] [20:18:15] - There are 2 tasks still running on 10.10.10.138
[*] Task tcp/80/whatweb on 10.10.10.138 finished successfully in 36 seconds
[*] [20:19:15] - There is 1 task still running on 10.10.10.138
[*] Service detection nmap-full-tcp on 10.10.10.138 finished successfully in 2 minutes, 51 seconds
[*] Found tcpwrapped on tcp/80 on target 10.10.10.138
[*] Running task tcp/80/sslscan on 10.10.10.138
[*] Task tcp/80/sslscan on 10.10.10.138 finished successfully in less than a second
[*] Finished scanning target 10.10.10.138 in 2 minutes, 52 seconds
[*] Finished scanning all targets in 2 minutes, 52 seconds!
```

It immediately found a webserver and SSH.

## nmap

I checked out the nmap output from autorecon:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup/results/10.10.10.138/scans]
└─$ cat _full_tcp_nmap.txt 
# Nmap 7.91 scan initiated Thu Jul 15 20:17:16 2021 as: nmap -vv --reason -Pn -A --osscan-guess --version-all -p- -oN /home/mac/.config/AutoRecon/results/10.10.10.138/scans/_full_tcp_nmap.txt -oX /home/mac/.config/AutoRecon/results/10.10.10.138/scans/xml/_full_tcp_nmap.xml 10.10.10.138
Nmap scan report for 10.10.10.138
Host is up, received user-set (0.016s latency).
Scanned at 2021-07-15 20:17:19 BST for 164s
Not shown: 65533 filtered ports
Reason: 65533 no-responses
PORT   STATE SERVICE    REASON  VERSION
22/tcp open  ssh        syn-ack OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 dd:53:10:70:0b:d0:47:0a:e2:7e:4a:b6:42:98:23:c7 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDKBbBK0GkiCbxmAbaYsF4DjDQ3JqErzEazl3v8OndVhynlxNA5sMnQmyH+7ZPdDx9IxvWFWkdvPDJC0rUj1CzOTOEjN61Qd7uQbo5x4rJd3PAgqU21H9NyuXt+T1S/Ud77xKei7fXt5kk1aL0/mqj8wTk6HDp0ZWrGBPCxcOxfE7NBcY3W++IIArn6irQUom0/AAtR3BseOf/VTdDWOXk/Ut3rrda4VMBpRcmTthjsTXAvKvPJcaWJATtRE2NmFjBWixzhQU+s30jPABHcVtxl/Fegr3mvS7O3MpPzoMBZP6Gw8d/bVabaCQ1JcEDwSBc9DaLm4cIhuW37dQDgqT1V
|   256 37:2e:14:68:ae:b9:c2:34:2b:6e:d9:92:bc:bf:bd:28 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPzrVwOU0bohC3eXLnH0Sn4f7UAwDy7jx4pS39wtkKMF5j9yKKfjiO+5YTU//inmSjlTgXBYNvaC3xfOM/Mb9RM=
|   256 93:ea:a8:40:42:c1:a8:33:85:b3:56:00:62:1c:a0:ab (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEuLLsM8u34m/7Hzh+yjYk4pu3WHsLOrPU2VeLn22UkO
80/tcp open  tcpwrapped syn-ack
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jul 15 20:20:04 2021 -- 1 IP address (1 host up) scanned in 168.95 seconds

```

It found two ports:
- SSH running on port 22 - this reveals the box to be a Debian machine
- Port 80 is open, but `tcpwrapped` suggests it's behind some form of firewall

There wasn't very much information, so I checked the specific port 80 scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup/results/10.10.10.138/scans]
└─$ cat tcp_80_http_nmap.txt 
# Nmap 7.91 scan initiated Thu Jul 15 20:17:44 2021 as: nmap -vv --reason -Pn -sV -p 80 "--script=banner,(http* or ssl*) and not (brute or broadcast or dos or external or http-slowloris* or fuzzer)" -oN /home/mac/.config/AutoRecon/results/10.10.10.138/scans/tcp_80_http_nmap.txt -oX /home/mac/.config/AutoRecon/results/10.10.10.138/scans/xml/tcp_80_http_nmap.xml 10.10.10.138
Nmap scan report for 10.10.10.138
Host is up, received user-set (0.021s latency).
Scanned at 2021-07-15 20:17:55 BST for 0s

PORT   STATE  SERVICE REASON       VERSION
80/tcp closed http    conn-refused

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jul 15 20:17:56 2021 -- 1 IP address (1 host up) scanned in 12.58 seconds
```

Maybe it isn't autorecon's fault - the connection is refused, but I can reach [[Writeups/Hack the Box/Boxes/Writeup/10 - Website|the site]] in browser, so perhaps it is rejecting the packets because of the nmap user agent.

I tried again later, after finding out about the Web Application Firewall on the box:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ nmap -p 80 -sC -sV 10.10.10.138
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-15 20:47 BST
Nmap scan report for writeup.htb (10.10.10.138)
Host is up (0.060s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/writeup/
|_http-title: Nothing here yet.

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.60 seconds
```

This revealed `robots.txt` and the writeup directory, by which point i'd already found.