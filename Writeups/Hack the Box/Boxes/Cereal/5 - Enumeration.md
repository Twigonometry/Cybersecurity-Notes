# Enumeration

## Autorecon

```bash
$ autorecon 10.10.10.217
[*] Scanning target 10.10.10.217
[*] Running service detection nmap-full-tcp on 10.10.10.217
[*] Running service detection nmap-quick on 10.10.10.217
[*] Running service detection nmap-top-20-udp on 10.10.10.217
[*] Service detection nmap-quick on 10.10.10.217 finished successfully in 24 seconds
[*] Found ssh on tcp/22 on target 10.10.10.217
[*] Found http on tcp/80 on target 10.10.10.217
[*] Found ssl/http on tcp/443 on target 10.10.10.217
[*] Running task tcp/22/sslscan on 10.10.10.217
[*] Running task tcp/22/nmap-ssh on 10.10.10.217
[*] Running task tcp/80/sslscan on 10.10.10.217
[*] Running task tcp/80/nmap-http on 10.10.10.217
[*] Running task tcp/80/curl-index on 10.10.10.217
[*] Running task tcp/80/curl-robots on 10.10.10.217
[*] Running task tcp/80/wkhtmltoimage on 10.10.10.217
[*] Running task tcp/80/whatweb on 10.10.10.217
[*] Task tcp/22/sslscan on 10.10.10.217 finished successfully in less than a second
[*] Task tcp/80/sslscan on 10.10.10.217 finished successfully in less than a second
[*] Running task tcp/80/nikto on 10.10.10.217
[*] Running task tcp/80/gobuster on 10.10.10.217
[*] Task tcp/80/curl-robots on 10.10.10.217 finished successfully in 1 second
[*] Task tcp/80/curl-index on 10.10.10.217 finished successfully in 1 second
[*] Running task tcp/443/sslscan on 10.10.10.217
[*] Running task tcp/443/nmap-http on 10.10.10.217
[!] Task tcp/80/gobuster on 10.10.10.217 returned non-zero exit code: 1
[*] Running task tcp/443/curl-index on 10.10.10.217
[*] Task tcp/443/curl-index on 10.10.10.217 finished successfully in 1 second
[*] Running task tcp/443/curl-robots on 10.10.10.217
[*] Task tcp/443/curl-robots on 10.10.10.217 finished successfully in less than a second
[*] Running task tcp/443/wkhtmltoimage on 10.10.10.217
[*] Task tcp/22/nmap-ssh on 10.10.10.217 finished successfully in 7 seconds
[*] Running task tcp/443/whatweb on 10.10.10.217
[*] Task tcp/80/wkhtmltoimage on 10.10.10.217 finished successfully in 16 seconds
[*] Running task tcp/443/nikto on 10.10.10.217
[*] Task tcp/443/wkhtmltoimage on 10.10.10.217 finished successfully in 13 seconds
[*] Running task tcp/443/gobuster on 10.10.10.217
[!] Task tcp/443/gobuster on 10.10.10.217 returned non-zero exit code: 1
[*] Task tcp/443/whatweb on 10.10.10.217 finished successfully in 22 seconds
[*] Task tcp/80/whatweb on 10.10.10.217 finished successfully in 30 seconds
[*] [15:42:32] - There are 7 tasks still running on 10.10.10.217
[*] Task tcp/443/nmap-http on 10.10.10.217 finished successfully in 45 seconds
[*] Task tcp/80/nmap-http on 10.10.10.217 finished successfully in 1 minute, 22 seconds
[*] [15:43:32] - There are 5 tasks still running on 10.10.10.217
[*] Service detection nmap-full-tcp on 10.10.10.217 finished successfully in 2 minutes, 14 seconds
[*] Task tcp/443/sslscan on 10.10.10.217 finished successfully in 1 minute, 54 seconds
[*] [15:44:32] - There are 3 tasks still running on 10.10.10.217
[*] [15:45:32] - There are 3 tasks still running on 10.10.10.217
[*] Task tcp/80/nikto on 10.10.10.217 finished successfully in 3 minutes, 40 seconds
[*] [15:46:32] - There are 2 tasks still running on 10.10.10.217
[*] [15:47:32] - There are 2 tasks still running on 10.10.10.217
[*] [15:48:32] - There are 2 tasks still running on 10.10.10.217
[*] [15:49:32] - There are 2 tasks still running on 10.10.10.217
[*] Service detection nmap-top-20-udp on 10.10.10.217 finished successfully in 8 minutes, 58 seconds
[*] [15:50:32] - There is 1 task still running on 10.10.10.217
[*] [15:51:32] - There is 1 task still running on 10.10.10.217
[*] [15:52:32] - There is 1 task still running on 10.10.10.217
[*] [15:53:32] - There is 1 task still running on 10.10.10.217
[*] [15:54:32] - There is 1 task still running on 10.10.10.217
[*] Task tcp/443/nikto on 10.10.10.217 finished successfully in 13 minutes, 8 seconds
[*] Finished scanning target 10.10.10.217 in 13 minutes, 48 seconds
[*] Finished scanning all targets in 13 minutes, 48 seconds!
```

## Nmap

Here is the output of Autorecon's full TCP Nmap scan:

```bash
# Nmap 7.91 scan initiated Tue Mar 16 15:41:35 2021 as: nmap -vv --reason -Pn -A --osscan-guess --version-all -p- -oN /root/Documents/HTB/cereal/results/10.10.10.217/scans/_full_tcp_nmap.txt -oX /root/Documents/HTB/cereal/results/10.10.10.217/scans/xml/_full_tcp_nmap.xml 10.10.10.217
Nmap scan report for 10.10.10.217
Host is up, received user-set (0.022s latency).
Scanned at 2021-03-16 15:41:37 GMT for 128s
Not shown: 65532 filtered ports
Reason: 65532 no-responses
PORT    STATE SERVICE  REASON          VERSION
22/tcp  open  ssh      syn-ack ttl 127 OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 08:8e:fe:04:8c:ad:6f:df:88:c7:f3:9a:c5:da:6d:ac (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDJ8WunqAHy9aWMuwZtw8rYXPpcWFOamTOdxvUDuFEzyvemSH8H8aPN3xVb8qhv6ZvSLW7gEDyNcu/+vPKo+G+Vy9sKyaFFdk7FiDgCIqnx5UyxPjZxBu6QxES8FndXmHoS3vifHcxBS3Y/e1Bx0MTLVfhWmBx7lJRpR4R7WHDgJ19yBsnB5921vNpVpSTzPV8eQI2lukoY/UMeatTLsB4SHqEljrUp3phY8YY6MHAWyVE0Ofp2xCiKhFwzfcl/kMEPSplrerse9MFCfpmD571vvzXiC9TKPajPdceVxKXJiBq6YjFE9gnBdmiiBVnGNZ735wiQe13GGvmEk9tuPAat
|   256 fb:f5:7b:a1:68:07:c0:7b:73:d2:ad:33:df:0a:fc:ac (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOv2yzt3CGzoXPn56DcYScZq9TapkXkNCTez76ygDDwAKBREa325DDx6ZDd99qtntl28Gzi1mZAfntdNulXmxqI=
|   256 cc:0e:70:ec:33:42:59:78:31:c0:4e:c2:a5:c9:0e:1e (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINFh4uMa9OjCINZ7M6/DSRhceOcHRP+n6o+py/ERV5fm
80/tcp  open  http     syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/plain).
443/tcp open  ssl/http syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-favicon: Unknown favicon MD5: 1A506D92387A36A4A778DF0D60892843
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/plain).
| ssl-cert: Subject: commonName=cereal.htb
| Subject Alternative Name: DNS:cereal.htb, DNS:source.cereal.htb
| Issuer: commonName=cereal.htb
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-11-11T19:57:18
| Not valid after:  2040-11-11T20:07:19
| MD5:   8785 41e5 4962 7041 af57 94e3 4564 090d
| SHA-1: 5841 b3f2 29f0 2ada 2c62 e1da 969d b966 57ad 5367
| -----BEGIN CERTIFICATE-----
| MIIDLjCCAhagAwIBAgIQYSvrrxz65LZHzBcVnRDa5TANBgkqhkiG9w0BAQsFADAV
| MRMwEQYDVQQDDApjZXJlYWwuaHRiMB4XDTIwMTExMTE5NTcxOFoXDTQwMTExMTIw
| MDcxOVowFTETMBEGA1UEAwwKY2VyZWFsLmh0YjCCASIwDQYJKoZIhvcNAQEBBQAD
| ggEPADCCAQoCggEBAMoaGpaAR2ALY//K4WkfjOPTXqfzIPio6lQpS2NOG9yMlDVT
| dYeFRwRyAxqgkGfNVchuKjnyc9BeJqILLyYDn5aK7/pIKc7bAPTs7B2YQpQXUTmH
| nVuP0JHMhflzDCMigr5XuZ7/xXh2fZbSantK/1PqeilClmjunoNBTsFHhNrb7XfK
| 2fwQDB0QS8TvLmcVKwx+qGt8Mtod165LUe6LPc1dK8tO5AxVGFoqE9w7jDa+QwK8
| eCazu5S7AV9TvInJrniz58fZ8zbJB4c2CQOB6BtFF9f3tft4pjAlToDifVZ0BMEl
| uTwpZFc8YxXNb0taTWSBTIpowL3RhZ3zmlmsebkCAwEAAaN6MHgwDgYDVR0PAQH/
| BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcDATAoBgNVHREEITAf
| ggpjZXJlYWwuaHRighFzb3VyY2UuY2VyZWFsLmh0YjAdBgNVHQ4EFgQU6pyk6xnL
| i8gMA3lTOcCaV3zlFP8wDQYJKoZIhvcNAQELBQADggEBAAUQw2xrtdJavFiYgfl8
| NN6fA0jlyqrln715AOipqPcN6gntAynC378nP42nr02cQCoBvXK6vhmZKeVpviDv
| pO9udH/JB0sKmCFJC5lQ3sHnxSUExBk+e3tUpiGGgKoQnCFRRBEkOTE3bI0Moam9
| Hd1OD32cp6uEmY7Nzhb6hYkR3S/MeYH78PvFZ430gLCFohc7aqimngSohAz8f+xc
| rS352J9a3+0TemS1KduwC/KFFG0o3ItDJSj4ypq9B6x2HGstfzmKzGqIu74Z5tXu
| guCIa2Jau8OdQ7K6aiPn39W+EnFLUQAMHqq7TZpxTb1SkV3hoVNvh63nxC1wyDrL
| iy0=
|_-----END CERTIFICATE-----
|_ssl-date: 2021-03-16T14:54:00+00:00; -49m45s from scanner time.
| tls-alpn: 
|_  http/1.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
TCP/IP fingerprint:
SCAN(V=7.91%E=4%D=3/16%OT=22%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=6050D231%P=x86_64-pc-linux-gnu)
SEQ(SP=102%GCD=1%ISR=108%II=I%TS=U)
OPS(O1=M54DNW8NNS%O2=M54DNW8NNS%O3=M54DNW8%O4=M54DNW8NNS%O5=M54DNW8NNS%O6=M54DNNS)
WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)
ECN(R=Y%DF=Y%TG=80%W=FFFF%O=M54DNW8NNS%CC=Y%Q=)
T1(R=Y%DF=Y%TG=80%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=N)
U1(R=N)
IE(R=Y%DFI=N%TG=80%CD=Z)

Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=258 (Good luck!)
IP ID Sequence Generation: Busy server or unknown class
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -49m45s

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   22.91 ms 10.10.14.1
2   22.74 ms 10.10.10.217

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Mar 16 15:43:45 2021 -- 1 IP address (1 host up) scanned in 133.41 seconds
```

Key findings:
- there are just a few ports open:
	- 22 for SSH
	- 80 for HTTP - Running a Microsoft IIS Server
	- 443 for HTTPS
- OpenSSH shows this is a Windows 7.7 box
- The SSL certificate exposes two domains, `cereal.htb` and `source.cereal.htb`

## Gobuster

An initial scan of the `cereal.htb` domain reveals there is some sort of generic response code for non-existent pages, meaning Gobuster gets several false positives:

```bash
gobuster dir -u http://cereal.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://cereal.htb
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/03/16 16:05:18 Starting gobuster
===============================================================
Error: the server returns a status code that matches the provided options for non existing urls. http://cereal.htb/de74da20-2e95-4ad1-bcf1-6d35cd02ad52 => 307. To force processing of Wildcard responses, specify the '--wildcard' switch
root@kali:~/Documents/HTB/cereal/results/10.10.10.217# gobuster dir -u http://cereal.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt --wildcard
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://cereal.htb
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/03/16 16:05:55 Starting gobuster
===============================================================
/modules (Status: 307)
/.php (Status: 307)
/cgi-bin (Status: 307)
/images (Status: 307)
/admin (Status: 307)
/search (Status: 307)
/cache (Status: 307)
/.html (Status: 307)
/includes (Status: 307)
/templates (Status: 307)

....[continues until stopped]....
```

We can deal with this behaviour by setting the `307` response code as a blacklisted response. Doing so overwrites the usual behaviour, so we also have to blacklist `404`:

```bash
┌──(mac㉿kali)-[~]
└─$ gobuster dir -u http://cereal.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -b 307,404
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://cereal.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   307,404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/05 18:35:39 Starting gobuster in directory enumeration mode
===============================================================
                                
===============================================================
2021/06/05 18:37:23 Finished
===============================================================
```

However, nothing was found.

### source.cereal.htb

```bash
┌──(mac㉿kali)-[~/Documents/HTB/cereal]
└─$ gobuster dir -u http://source.cereal.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt --wildcard -s 200,301,302
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://source.cereal.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/05 18:39:38 Starting gobuster in directory enumeration mode
===============================================================
/aspnet_client        (Status: 301) [Size: 162] [--> http://source.cereal.htb/aspnet_client/]
/uploads              (Status: 301) [Size: 156] [--> http://source.cereal.htb/uploads/]      
/.                    (Status: 500) [Size: 10090]                                            
/.git                 (Status: 301) [Size: 153] [--> http://source.cereal.htb/.git/]         
                                                                                             
===============================================================
2021/06/05 18:41:22 Finished
===============================================================
```

Crucially, this finds a `/.git` directory and a `/uploads` directory.