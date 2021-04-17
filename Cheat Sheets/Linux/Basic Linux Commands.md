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

## Switch User

The `su` command switches to another user on the machine. You must know their password.

This command will attempt to switch to the `jane` user:

```bash
$ su jane
```

If no username is provided, `su` will attempt to login as root:

```bash
$ su
```

Using the `-` flag simulates a real login - you will often see `su -` in writeups as shorthand for `su root`:

```bash
$ su -
```

However this command is effectively the same as the one above, apart from a few [small differences](https://unix.stackexchange.com/questions/15611/what-is-the-difference-between-su-and-su-root) like the clearing of environment variables.

## grep

`grep` is an incredibly useful command that allows searching the contents of files and the outputs of terminal commands. 

**Grepping a File**

```bash
$ grep [SEARCH_TERM] /path/to/file
```

or, using pipe (`|`):

```bash
$ cat /path/to/file | grep [SEARCH_TERM]
```

**Grepping Terminal Output**

You can use the pipe operator to run `grep` on the output of any command. A common use case is searching the output of a directory.

```bash
$ ls -la | grep [SEARCH_TERM]
```

## awk
`awk` can be used for manipulating structured text and extracting specific fields. Think of it as a way of selecting a specific column from a load of structured data.

For example, a list of employee names:

```bash
$ cat employees
John Doe
Jane Doe
```

`awk` could be used to extract the first names of these employees:

```bash
$ cat employees | awk '{print $1}'
```

Where `$x` represents the `xth` column (1-indexed).

You can specify a different 'field separator' with the `-F` flag:

```bash
$ cat employees
John:Doe
Jane:Doe
$ cat employees | awk -F ':' '{print $1}'
John
Jane
```