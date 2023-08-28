# Networking

## Get your IP

Use this command to see your IP address. `ifconfig` on its own will list all interfaces, or you can specify one.

```bash
ifconfig [interface]
```

specify `tun0` as the interface on HacktheBox

## Curl

Curl POST Request:

```bash
curl -X POST [options] [URL]
```

## SSH

### Generating a Keypair

Generate a public and private key with the following command. By default it will save to the `~/.ssh` directory.

```bash
$ ssh-keygen
```

You can also specify a key name with the `-f` flag. By default the private key will be called `id_rsa` and the public key will be called `id_rsa.pub`.

```bash
$ ssh-keygen -f key_name
```

### Secure Copy
```bash
$ scp [OPTION] [[user@]SRC_HOST:]file1 [[user@]DEST_HOST:]file2
```

## Testing a Connection

How to test if you can force a box to communicate back to you?

### Netcat

A `netcat` listener is useful for many reasons, including catching shells. But it can also be good for testing a one-off connection, such as a HTTP request.

This is the basic syntax for a netcat listener:

```bash
$ nc -lp [PORT]
```

This will catch incoming connections, and also harvest user agents from the requestor.

To start on a priveleged port, such as 80 or 443, you must run with root priveleges:

```bash
$ sudo nc -lnvp 80
```

For extra info, use the `-v` flag to put it in verbose mode. Use the `-n` flag to not perform DNS lookups on the incoming connections. This is the standard netcat command I use:

```bash
$ nc -lnvp [PORT]
```

Netcat can listen for anything sent over TCP or UDP (practically any web traffic besides `ping` requests).

Netcat closes after one connection by default. To have your netcat listener stay open after a connection, use the `-k` flag.

### Python webserver

A Python webserver is better than Netcat if you want to make multiple requests, as it stays open after a connection by default. However, a Python webserver does not harvest as much information by default, such as User Agent strings

On host machine, start a webserver:

```bash
$ python3 -m http.server [PORT]
```

(*Note:* This listens on port 8000 by default. See more details at [[Fundamental Skills#Filesharing Between Computers]])

Then make a connection to your box, using any method available to you:
- Curl: `$ curl http://[IP]:[PORT]`
- Wget: `$ wget http://[IP]:[PORT]`
- Netcat: `$ nc [IP]:[PORT]`
- Curl.exe (Windows 10): `curl.exe --url http://[IP]:[PORT]`
- Powershell: `powershell -command "Invoke-WebRequest -Uri http://[IP]:[PORT]"`

See the following [Stack Overflow Post](https://serverfault.com/questions/483754/is-there-a-built-in-command-line-tool-under-windows-like-wget-curl) for more windows options.

### Ping

For non-TCP/UDP requests, start a tcpdump and send a ping.

On the host machine, start a listener for ICMP (ping) requests:

```bash
┌──(mac㉿kali)-[~]
└─$ sudo tcpdump -i [INTERFACE] -n icmp
[sudo] password for mac: 
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
```

On the target machine, ping your box once using either:
- `ping -c 1 [IP]` (Linux)
- `ping -n 1 [IP]` (Windows)

Pinging only once is necessary if you have non-interactive remote code execution (i.e. you can issue a command, but not press `Ctrl + C` to abort it). Otherwise, the box will continue pinging forever.

## PHP Webservers

Similarly to the Python Webserver, this starts a server locally and quickly, and gives you output logs. However, it is more useful for serving PHP files than it is for transferring files between boxes.

If you want to test a PHP script locally, you can run a PHP webserver in the serving directory like so:

```bash
$ php -S localhost:[PORT]
```

Like with Python, you will need root privileges to run on a priveleged port (<1000)

You can then access PHP files with curl, or in the browser. For example, testing a webshell:

```bash
$ curl http://localhost:[PORT]/cmdphp.php?cmd=id
```

PHP webservers can also come in handy when testing web applications on a remote box. If you setup an [[Linux Networking#With SSH|SSH tunnel]] to the remote server, you can run a PHP server to test it and see the output. This can help debug the server remotely.

A practical example of this can be seen at [[20 - Shell as roy#Testing the Web App]]

## Tunneling

Use tunneling to access a remote port via a local port.

For example, a webserver is running locally on a remote box (it is not exposed publicly to the internet). If this server was running on port 9999, it would be accessed on the remote box via `http://localhost:9999`.

If we have a way of tunneling to this box, we can set up a relay from our computer to the remote computer. For example, we could tell our port 8000 to point to the remote server's port 9999. If the remote server's IP was `192.0.0.2`, the tunnel would look like this:
- We make a request to `http://localhost:8000` on our local machine
- This request is forwarded, via a tunnel, to `http://192.0.0.2:9999`

This setup allows us, for example, to access a website in the browser that we may only have been able to interact with previously over command line.

### With SSH

If we have SSH access to this machine, we can use SSH tunneling to accomplish this goal:

```bash
$ ssh -L 8000:localhost:9999 [USER@]192.0.0.2 [-i /path/to/private/key]
```

This assumes the same addresses as above. It will ask us to log in, or use an optional provided private key. Once the SSH connection completes, an SSH terminal will open, and the tunnel will be setup.

You can then 'navigate' to the remote host's 'local' webserver by visiting your own localhost:

```bash
$ curl localhost:8000
```

A good guide to this process can be found at [Linux Hint](https://linuxhint.com/ssh-port-forwarding-linux/), which includes SSH setup instructions.

# Tags

#cheat-sheet #unix #networking