# Basic Linux Commands

*Note*: In these, and all other examples in the cheatsheets, the `$` prefix before a command indicates the use of a linux command line input. It is not part of the command - it is there to show you which lines are input, and which are output (lines without a `$` prefix)

## Check Current Working Directory

Where are you on this file system? Find out with `pwd`:

```bash
$ pwd
/var/www/html
```

Useful just after landing a shell or if you get lost.

## Change Directory

The `cd` command changes directory:

```bash
$ pwd
/home/mac
$ cd /opt
$ pwd
/opt
```

Go to your home directory:

```bash
$ cd ~
```

Go back to the previous directory:

```bash
$ cd -
```

Go to the root of the filesystem:

```bash
$ cd /
```

## List Files

List files in the current working directory:

```bash
$ ls
```

List all files, including hidden ones (preceded by a `.`, e.g. `.env`):

```bash
$ ls -a
```

Display extra information about files, including timestamps and file ownership:

```bash
$ ls -l
```

Combine the two:

```bash
$ ls -la
```