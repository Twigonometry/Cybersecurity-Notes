# To Add
This note lays out a roadmap for content I want to add to this repository. You can keep track of my plans here.

If you want to suggest some content to add and it isn't on the list, feel free to [open an issue](https://github.com/Twigonometry/Cybersecurity-Notes/issues).

## Fundamental Skills

The roadmap for content I want to add to Fundamental Skills section

- [ ] Burp Suite
	- [ ] On the fly editing
	- [ ] Repeater
		- [ ] Things to look for
		- [ ] Multiple Tabs
		- [ ] Inspecting Content size to see if page contents change
	- [ ] Intruder
- [ ] Web Requests
	- [ ] Basics of HTTP requests
		- [ ] Request methods
			- [ ] CRUD
		- [ ] Response codes
- [ ] Web Authentication Methods
	- [ ] Cookies
	- [ ] JWT
- [ ] Fuzzing
	- [ ] Wfuzz
	- [ ] FFUF
	- [ ] Fuzzing APIs
	- [ ] Attaching authentication to fuzzer
	- [ ] Trying to provoke errors
	- [ ] Polyglots
- [ ] Password Cracking
	- [ ] Recognising hash types
	- [ ] Recognising other encodings - b64, ROT
- [ ] Recognising common data types
	- [ ] JSON
		- [ ] Using JQ
	- [ ] Encoding and decoding b64 in command line
- [ ] Password spraying
	- [ ] Using CME
	- [ ] Using hydra
- [ ] File transfers
- [ ] Shells
	- [ ] Web shells
	- [ ] Reverse shells
	- [ ] Catching shells
		- [ ] Netcat
		- [ ] Multi handler
	- [ ] Generating shellcode
- [ ] Searching for CVEs
	- [ ] Looking for version numbers
		- [ ] Websites
		- [ ] Nmap OS versions - how to read patch numbers
	- [ ] Searchsploit
		- [ ] Mirroring code
		- [ ] Editing exploits
			- [ ] Common pitfalls e.g. python2
			- [ ] Tools e.g. 2to3
- [ ] General hacking methodology steps
	- [ ] Overview of steps
		- [ ] Recon
		- [ ] Looking for a foothold
			- [ ] RCE
			- [ ] File read/write
			- [ ] Auth bypass
	- [ ] Link to main sheet
- [ ] Exploiting the OWASP top 10
	- [ ] Identifying them
	- [ ] Brief overview of all
- [ ] Terminal basic functionality and productivity
	- [ ] `wc`
	- [ ] `grep`
	- [ ] `less`
	- [ ] pipes
	- [ ] redirects

For each topic, include:
- 'Read More' link, linking reader to main cheatsheet
- 'Practical Examples' link, linking either to specific boxes and examples, or to a practical examples section in the corresponding cheatsheet if it exists
- 'Where to Practice This Skill' section, which links places you can test the skill - including specific HTB boxes, THM Rooms, or VulnHub VMs

## Methodology

- [ ] Flowchart
- [ ] Checklist
	- [ ] Subdomain bruteforcing with wfuzz
	- [ ] Scanning - nmap standard scripts with OS enum, all ports, UDP, vuln
	- [ ] Gobuster for files and for directories (-f)
	- [ ] Script all of this/add it to autorecon profile
	- [ ] Look for shellshock, log poisoning
	- [ ] Trying default credentials
- [ ] Add everything from "key lessons" sections

## Setting stuff up

- [ ] VMs
	- [ ] Networking VMs
	- [ ] Increasing partition size
	- [ ] Installing VSCode
- [ ] Nmap privileged setup
- [ ] Configuring proxychains

## Cheatsheets on Common Tools

- [ ] Gobuster
	- [ ] Add links to practical examples
	- [x] VHOST busting
- [ ] SQLMap
	- [ ] Blind Injection
- [ ] CrackMapExec
	- [ ] Password Sprays
- [ ] Impacket
	- [ ] Secrets Dump
- [ ] Kerbrute
	- [ ] Basic pre-auth attack
	- [ ] Password Spray
- [ ] rpcclient
	- [ ] Null auth
- [ ] Netcat
	- [x] Basic syntax
	- [ ] File transfer
	- [ ] Traditional mode
- [ ] Mimikatz
- [ ] WPScan
- [ ] WFuzz
	- [ ] Subdomain Bruteforcing
- [ ] FFUF

## Guides to Basic Linux Commands
- [ ] awk
- [ ] grep
	- [x] Basic Syntax
	- [ ] Regex

## Common Services and Protocols

- [ ] FTP
	- [ ] Anonymous Authentication
	- [ ] Basic get/upload syntax
	- [ ] Uploading malicious payloads via FTP
	- [ ] Exploits on old versions
- [ ] SMB
	- [ ] Anonymous Authentication
	- [x] Enumeration
- [ ] LDAP
	- [ ] Basic query structure

## Password Cracking

- [ ] Cracking encrypted files
	- [ ] zip2john
	- [ ] keepass2john

## XSS

- [ ] XSS Payload Lists
	- [ ] `XMLHttpRequest()`
	- [ ] `fetch()`
	- [ ] Cookie stealing
	- [ ] Redirects
- [ ] Obfuscation Examples
- [ ] Interesting Delivery Methods
	- [ ] Markdown
- [ ] Troubleshooting
	- [ ] `img` source callbacks

## Shells

- [ ] Powershell Shells
- [ ] ASP Shells
- [ ] PHP Shells
- [ ] Netcat Shells
- [x] Bash Shells

## Detailed Writeups

- [ ] Deserialisation
	- [ ] .NET Deserialisation
	- [ ] PHP Deserialisation
	- [ ] Deserialisation on Laboratory
- [ ] Web assessment methodology
	- [ ] Enumerating Services
	- [ ] Finding Version numbers
- [ ] Live overflow Web server hosting advice
	- [ ] Add practical examples of implementing the advice
- [ ] Kerberos
- [ ] Crypto
	- [ ] Hmac
	- [ ] Extension attacks
- [ ] WiFi hacking
	- [ ] Deauth
	- [ ] brooks notes
- [ ] DNS enumeration
	- [ ] Read DNS sec
	- [ ] Over tor
	- [ ] Burp + Tor proxy at same time
	- [ ] proxychains

## Writeups

- [ ] HTB
	- [x] Bucket
	- [ ] Time
	- [ ] Doctor
	- [ ] Academy
	- [ ] Traceback
	- [ ] OpenAdmin
	- [ ] Laboratory
	- [ ] Passage
	- [ ] Tenet
- [ ] PwnED CTF
- [ ] SESH CTF
	- [ ] Challenge Development
		- [ ] Saucy Part 3
		- [ ] Path to Root
		- [ ] General AWS and Docker deployment
	- [ ] Writeups
		- [x] Saucy Part 3
		- [x] Path to Root
		- [ ] Robots Challenges

## Reconnaisance

- [ ] Options for scanning a range of targets
- [ ] Options for scanning targets with an unknown address

## Enumeration

- [ ] General
	- [ ] Linpeas
		- [ ] What to look for
		- [ ] What to disregard
	- [ ] Winpeas
- [ ] Manual Versions of Common Enumeration Scripts
	- [ ] Linpeas
		- [x] Details of auto exploit OSCP disqualification
			- [x] Tweet
			- [x] OSCP statement
			- [x] Identify offending code and show how to remove it
		- [ ] How to replicate most common commands
			- [ ] Listing services
			- [ ] Finding passwords
	- [ ] manual port scanning
		- [ ] Bashscan

## Misc

- [ ] Tips and Tricks
	- [ ] Port Knocking
- [ ] Add instructions for how to contribute to repo
- [ ] Link preview hacks

## Linux Operating System

- [ ] File permissions
	- [ ] rwx
	- [ ] ACL
	- [ ] Suid
- [ ] Credentials
	- [ ] Shadow File
- [ ] Sudo
	- [ ] Sudoers file
	- [ ] Sudo Visudo CTF practical example

## Windows Operating System

- [ ] File Permissions
- [ ] Credentials
	- [ ] Where are credentials stored
	- [ ] Exploits involving registry
- [ ] Powershell secure strings

## CVEs

General
- [ ] Add titles to all
- [ ] Add PoCs both with and without msf

Specific CVEs
- [x] Eternal Blue
- [ ] From Cyber Awareness Day
	- [ ] Android ADB - CVE-2019-16273
	- [ ] Teams RCE - CVE-2020-1709
- [ ] From Articles
- [ ] Smarty CVEs

## AWS

- [ ] General AWS
	- [ ] Cloudformation
- [ ] Security
	- [ ] Bucket policies
	- [ ] Enumerating S3 Buckets
	- [ ] Scanning for misconfigured buckets

## From Old SESH Sessions
- [ ] Kerberoasting
- [ ] BOF

## OWASP Top Ten

- XSS
	- [ ] Tricks
	- [ ] Payloads
	- [ ] Fetch
	- [ ] XMLHTTP
	- [ ] Img source callbacks
	- [ ] Markdown
	- [ ] Delivery methods
	- [ ] Defence evasion
	- [ ] Dangerous functions like eval
- [ ] Deserialisation
	- [ ] Python + flask
	- [ ] PHP
	- [ ] .NET
	- [ ] Java