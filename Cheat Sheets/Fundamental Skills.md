# Fundamental Skills
Here are some of the things I consider fundamental to CTFs, Hack the Box-esque challenges, and web hacking.

Skills include knowing how to transfer files between computers, crack passwords, remotely access a machine, run reconnaisance scans, analyse HTTP requests, and script with bash or python.

Each skill links to a more detailed entry in the cheatsheets, and some even have links to practical example where the skills have been used on a box or challenge.

Start here if you are a complete beginner!

## Remote Access to a Computer

### SSH

### RDP

## Filesharing Between Computers

### FTP

### Netcat

### 

## Shells

**What is a shell?**

**When might you want to use a shell?** 

### Web Shells

### Bash Reverse Shells

## Network Requests

**What does a HTTP Request look like?**

### Sending Requests with Curl

### Manipulating Requests with Burp Suite

### More Details

See [[Sending HTTP Requests]]

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

### Python

### Powershell