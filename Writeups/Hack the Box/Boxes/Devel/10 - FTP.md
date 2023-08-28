# FTP

My first port of call was checking out FTP.

We can do anonymous login:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/devel/ftp]
└─$ ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:mac): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
```

If we make a file locally, we can put it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/devel]
└─$ mkdir ftp
┌──(mac㉿kali)-[~/Documents/HTB/devel]
└─$ echo "test" > test
┌──(mac㉿kali)-[~/Documents/HTB/devel]
└─$ mv test ftp/
```

On the FTP client:

```
?Invalid command
ftp> put test
local: test remote: test
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
6 bytes sent in 0.09 secs (0.0625 kB/s)
```

So we have arbitrary upload permissions. Let's check out the site to see if we can exploit this.

*Note*: we don't need to be in the `ftp` directory when we launch the client - we can connect to FTP then use `lcd ftp` also.