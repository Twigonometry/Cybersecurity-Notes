# Enumeration

## nmap

We started with an `nmap` scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame]
└─$ nmap -sC -sV -v -Pn -oA nmap/ 10.10.10.3
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-03 19:35 BST
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 19:35
Completed NSE at 19:35, 0.00s elapsed
Initiating NSE at 19:35
Completed NSE at 19:35, 0.00s elapsed
Initiating NSE at 19:35
Completed NSE at 19:35, 0.00s elapsed
Initiating Parallel DNS resolution of 1 host. at 19:35
Completed Parallel DNS resolution of 1 host. at 19:35, 0.00s elapsed
Initiating Connect Scan at 19:35
Scanning 10.10.10.3 [1000 ports]
Discovered open port 21/tcp on 10.10.10.3
Discovered open port 445/tcp on 10.10.10.3
Discovered open port 22/tcp on 10.10.10.3
Discovered open port 139/tcp on 10.10.10.3
Completed Connect Scan at 19:35, 4.74s elapsed (1000 total ports)
Initiating Service scan at 19:35
Scanning 4 services on 10.10.10.3
Completed Service scan at 19:36, 11.19s elapsed (4 services on 1 host)
NSE: Script scanning 10.10.10.3.
Initiating NSE at 19:36
NSE: [ftp-bounce] PORT response: 500 Illegal PORT command.
Completed NSE at 19:36, 40.10s elapsed
Initiating NSE at 19:36
Completed NSE at 19:36, 0.16s elapsed
Initiating NSE at 19:36
Completed NSE at 19:36, 0.00s elapsed
Nmap scan report for 10.10.10.3
Host is up (0.023s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.13
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h04m15s, deviation: 2h49m43s, median: 4m14s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2021-05-03T14:40:27-04:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

NSE: Script Post-scanning.
Initiating NSE at 19:36
Completed NSE at 19:36, 0.00s elapsed
Initiating NSE at 19:36
Completed NSE at 19:36, 0.00s elapsed
Initiating NSE at 19:36
Completed NSE at 19:36, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 57.09 seconds
```

We have an FTP server, with anonymous login allowed. It also exposed a version name, `vsFTPd 2.3.4`.

We also have samba and SSH.

The scan exposed a domain name: `lame.hackthebox.gr`, and that the box was running on Ubuntu.

## Extra nmap

I set off a full port scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame]
└─$ nmap -p- -Pn -oA nmap/all-ports 10.10.10.3
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-03 21:18 BST
Nmap scan report for 10.10.10.3
Host is up (0.023s latency).
Not shown: 65530 filtered ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3632/tcp open  distccd

Nmap done: 1 IP address (1 host up) scanned in 104.46 seconds
```

This returned one extra port, running `distccd`. I didn't exploit this, but [0xdf](https://0xdf.gitlab.io/2020/04/08/htb-lame-more.html) has a nice writeup.

I also ran a vuln scan:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/lame]
└─$ nmap --script vuln -Pn -oA nmap/vuln 10.10.10.3
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-03 19:40 BST
Nmap scan report for 10.10.10.3
Host is up (0.021s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE
21/tcp  open  ftp
|_sslv2-drown: 
22/tcp  open  ssh
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: false
|_smb-vuln-regsvc-dos: ERROR: Script execution failed (use -d to debug)

Nmap done: 1 IP address (1 host up) scanned in 86.61 seconds
```

This gave nothing interesting.

# Tags

#writeup #oscp-prep #enum 