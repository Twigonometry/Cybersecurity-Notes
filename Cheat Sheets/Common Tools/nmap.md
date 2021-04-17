# nmap
A tool for discovering what services are running on each port for a given host

The repository can be found at [https://github.com/nmap/nmap](https://github.com/nmap/nmap)

## Installation

The official nmap page has [extensive installation instructions](https://nmap.org/book/install.html) if you cannot find what you need below. I have covered the most common use cases.

### Kali Linux

`nmap` comes preinstalled on Kali Linux

### Other Linux Distributions

`nmap` has an `apt` package, and can be installed with:

```bash
$ sudo apt install nmap
```

To check your distribution has this package, run `apt search nmap`

### From Source

You can install `nmap` from source, by following the repository's [install instructions](https://github.com/nmap/nmap#installing)

## Basic Syntax

**Scan a Target**

For a target with IP address `a.b.c.d`, run the following command:

```bash
$ nmap a.b.c.d
```

By default, `nmap` scans the top 1000 most common ports

**General Purpose nmap Command**

I run this command when I first enumerate most targets. It shows ports as it finds them with the `-v` flag, tries to enumerate service versions and runs some default scripts, and then outputs the results to the `nmap/` directory, which you must make first (`mkdir nmap`):

```bash
$ nmap -v -sC -sV -oA nmap/ a.b.c.d
```

**Troubleshooting**

If your probes are dropped, try running `nmap` with the `-Pn` flag:

```bash
$ nmap -Pn a.b.c.d
```

See what `nmap` is doing by putting it in verbose mode with the `-v` flag. This will let you see ports as it discovers them, and troubleshoot issues.

```bash
$ nmap -v a.b.c.d
```

## Extra Information

`nmap` is capable of running several scripts for enumerating extra information about a service. Specify the `-sC` flag to run default scripts:

```bash
$ nmap -sC a.b.c.d
```

`nmap` can also enumerate the versions of software running on a port. Use the `-sV` flag to do this:

```bash
$ nmap -sV a.b.c.d
```

## Specifying Ports

**Scan a Single Port**

To scan port X, run this command:

```bash
$ nmap -p X a.b.c.d
```

**Scan a Range of Ports**

To scan ports X to Y (numerically), run this command:

```bash
$ nmap -p X-Y a.b.c.d
```

**Scan All Ports**

To scan all 65535 ports, run this command:

```bash
$ nmap -p- a.b.c.d
```

It is good practice to run this command in the background after your initial `nmap` scan of the top 1000 ports. You can do this with a sleep, waiting 5 minutes, like so:

```bash
$ sleep 300; nmap -v -p- -oA nmap/target-all-ports a.b.c.d
```

This will save the output to the same directory as in the **General Purpose nmap Command** above, with a different suffix.

It is unnecessary to run scripts on this scan - it is just useful for finding open ports you may have missed, and you can re-enumerate them with scripts afterwards by specifying those ports.

## Scan UDP

Some services run only over UDP, and may be missed by a standard TCP scan. Scan UDP with the `-sU` flag:

```bash
$ nmap -sU a.b.c.d
```

Practical Example: [Ippsec's Intense Video](https://youtu.be/nBg6zUalb7c?t=143)

## Adjusting Speed

Use the `--min-rate` flag to increase the speed of your scan. This adjusts the minimum number of packets sent per second. It is useful if you want to rapidly re-scan, perhaps with a full port scan or a UDP scan.

```bash
$ nmap --min-rate 10000 a.b.c.d
```

Only do this if you are not concerned about crashing the target, or concerned about Operational Security - a high minimum rate will stand out like a sore thumb in server logs and Intrusion Detection Systems (IDS).

Conversely, you can also set the `--max-rate` flag to put an upper limit on the number of requests per second. This is useful if you wish to evade detection.

```bash
$ nmap --max-rate 30 a.b.c.d
```

### Timing Templates

`nmap` provides a number of preset timing templates, that are chosen with the `-T` flag. These set a number of rate-related variables, including the ones above.

```bash
$ nmap -T paranoid|sneaky|polite|normal|aggressive|insane a.b.c.d
```

You can also use the numeric equivalents,  `-T0` through `-T5`. The [nmap documentation](https://nmap.org/book/man-performance.html) recommends `-T4` on a high broadband speed, and `-T1` or lower for IDS evasion.

You can also combine a template with a specific performance flag, such as a `--scan-delay` between probes to avoid some rate limiting defences:

```bash
$ nmap -T4 --scan-delay 1s a.b.c.d
```

This will keep all the other settings of `aggresive` mode, such as the lowered `max-retries`.

Full details on timing template values are available at [https://nmap.org/book/performance-timing-templates.html](https://nmap.org/book/performance-timing-templates.html)