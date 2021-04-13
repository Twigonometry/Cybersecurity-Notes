# Reconnaisance

## nmap

Usually the first step in reconnaisance, `nmap` allows you to enumerate services that are running on a target machine.

**Basic Syntax**

General purpose `nmap` scan for target IP `a.b.c.d`:

```bash
$ nmap -v -sC -sV -oA nmap/ a.b.c.d
```

In another terminal, run an all ports scan after five minutes has passed:

```bash
$ nmap -p- a.b.c.d
```

**More Details**

See the [[nmap|nmap Cheat Sheet]] for more details.

## Autorecon

Download the tool from the [Git Repository](https://github.com/Tib3rius/AutoRecon), where you can also see a whole host of extra usage details.

As a personal preference, I like doing my standard reconnaisance manually, and running autorecon in the background. This helps me process the information better as I find it.

It is particularly useful if you are prone to forgetting crucial steps, for example an all-ports `nmap` scan.

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