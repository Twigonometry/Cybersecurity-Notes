# Methodology Checklist

## Initial Enumeration

Use this enumeration to do initial scans of the network - the services discovered can then be subsequently enumerated.

### nmap

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

#### OS Enumeration

```bash
$ nmap -O -oA nmap/target-os [IP/HOST]
```

#### UDP

```bash
$ nmap -sU -oA nmap/target-udp [IP/HOST]
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