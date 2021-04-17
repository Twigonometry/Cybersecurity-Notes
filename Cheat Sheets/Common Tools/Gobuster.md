# Gobuster

A tool for brute forcing webpages (aka directory busting), DNS names, and virtual hosts - written in [Go](https://golang.org/).

The repository can be found at [https://github.com/OJ/gobuster](https://github.com/OJ/gobuster)

## Installation

### Kali Linux

`gobuster` does not come preinstalled on Kali Linux, but it does have a package. Install with:

```bash
$ sudo apt-get install gobuster
```

### Other Operating Systems

If your package manager doesn't have `gobuster`, or you want to install from source on Kali instead of using `apt`, you can follow the [installation instructions](https://github.com/OJ/gobuster#easy-installation) in the repo.

## Dir Busting

**Basic Syntax**

```bash
$ gobuster dir -u [URL] -w /path/to/wordlist
```

**Which Wordlist to Use?**

I like to use the [SecLists](https://github.com/danielmiessler/SecLists) Discovery lists. The most common one I use is located at `/path/to/seclists/Discovery/Web-Content/raft-small-words.txt`. SecLists comes preinstalled on Kali Linux, and is found at `/usr/share/seclists`.

You may also wish to use a larger list, such as `raft-large-words.txt`, or a list for a specific platform, such as `tomcat.txt` against a known Tomcat server.

If you do not wish to install SecLists some distributions come with alternative wordlists, for example `/usr/share/wordlists/dirb/common.txt`. However, many of the `dirb` wordlists miss important items, such as checking for a `.git` file.

**Add Extensions**

If you know your target site is using a specific file extension, such as `php` or `jsp`, you can specify this with the `-x` flag

```bash
$ gobuster dir -u example.com -w /path/to/wordlist -x php,asp
```