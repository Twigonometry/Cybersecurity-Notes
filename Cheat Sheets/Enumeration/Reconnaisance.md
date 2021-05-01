# Reconnaisance
These tools help you get a sense of what a target does and what services are running on it.

## nmap

I always start with an `nmap` scan. It is a tool for discovering open ports (communication points) on a network, and what services are running on them.

A good general purpose `nmap` command looks like this:

```bash
$ nmap -v -sC -sV -oA nmap/ [IP/HOSTNAME]
```

It shows ports as it finds them with the `-v` flag, tries to enumerate service versions and runs some default scripts, and then outputs the results to the `nmap/` directory, which you must make first (with `mkdir nmap`)

By default, `nmap` only scans the 1000 most common ports. To scan all ports, run this command:

```bash
$ nmap -p- -oA nmap/all-ports [IP/HOSTNAME]
```

It is best to do this after your first scan has completed.

See more details about nmap's various functions in the [[nmap|nmap cheatsheet]], including:
- Installation Instructions
- Scanning Specific Ports
- Scanning UDP Services
- Timing and Intrusion Detection Evasion

## Autorecon

This tool performs a number of automated scans. It starts with an `nmap` scan, and uses the results from that to launch a number of other scans, such as directory discovery with [[Gobuster]].

Download the tool from the [Git Repository](https://github.com/Tib3rius/AutoRecon), where you can also see a whole host of extra usage details.

As a personal preference, I like doing my standard reconnaisance manually, and running autorecon in the background. This helps me process the information better as I find it.

It is useful as it cannot forget crucial steps, for example an all-ports `nmap` scan.

**Basic Syntax**

```bash
autorecon [IP/Domain Name]
```

**Multiple Targets**

Specify multiple targets after you define your options (flags). E.g.

```bash
autorecon [OPTIONS] a.b.c.d foo.bar w.x.y.z
```

Or use CIDR Notation to define a range of targets:

```bash
autorecon [OPTIONS] a.b.c.d/24
```

Or use a target file with the `-t` flag:

```bash
autorecon -t /path/to/target_file a.b.c.d
```

**Output Options**

Set an output directory with the `-o` flag:

```bash
autorecon -o /path/to/output_dir example.com
```

### Running a Webserver

Once your scans are finished, you can easily view the results by starting a webserver in the `/results` directory and visiting it on localhost:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/BOX/results/]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Then visit `http://localhost:8000` and view the results in browser.

# Tags

#cheat-sheet #enum 