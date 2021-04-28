# smbclient

`smbclient` is a tool used to connect to Samba servers - Samba is a Windows-based filesharing protocol.

## Basic Syntax

Connect to a server:

```bash
$ smbclient //[IP]
```

Connect to a specific share:

```bash
$ smbclient //[IP]/[SHARE]
```

### Basic Commands

**File Upload and Download**

Use `get` to download a file to your local machine's current working directory:

```bash
smb: \> get [FILE]
```

Use `put` to upload a file from your local machine's current working directory:

```bash
smb: \> put [FILE]
```

**Navigating Samba**

Use `dir` to list files:

```bash
smb: \> dir
```

Use `cd` to change directory:

```bash
smb: \> cd [DIRECTORY_PATH]
```

Use `lcd` to change directory locally:

```bash
smb: \> lcd [LOCAL_DIRECTORY_PATH]
```

Use `exit` to exit the client:

```bash
smb: \> exit
```

## Download a File

Download a file from a samba share in one line:

```bash
$ smbclient '//[IP]/[SHARE]' -c 'lcd [DOWNLOAD_PATH]; cd [DIRECTORY]; get [FILENAME]'
```

The `-c` flag runs a series of commands. `lcd` changes directory on the local machine (i.e. your Linux host), so use this to specify where the downloaded file should land.

## Upload a File

Upload a file to a samba share in one line:

```bash
$ smbclient '//[IP]/[SHARE]' -c 'cd [REMOTE_PATH]; lcd [LOCAL_DIRECTORY]; put [LOCAL_FILENAME]'
```

Or, using `curl`:

```bash
$ curl --upload-file /path/to/file -u [USERNAME] smb://[IP]/[SHARE]
```

## List Shares

Use the `-L` flag to list shares:

```bash
$ smbclient -L hostname -U username
```

## Mount a Share

You can use the `mount` tool to mount a Samba share.

First, make a `/mnt` directory if you don't have one. Do this with root privileges:

```bash
$ sudo mkdir /mnt
```

Then make a subfolder in that directory and name it after the share you intend to mount:

```bash
$ sudo mkdir /mnt/[SHARE]
```

Then install `cifs-utils` if you don't already have them:

```bash
$ sudo apt install cifs-utils
```

These utilities will help you remotely mount SMB shares.

Then run this command to mount the share:

```bash
$ sudo mount -t cifs //[IP]/[SHARE] /mnt/[SHARE]
```

Hit enter when prompted for a password.

### Unmount the Share

Unmount the share if you don't need it anymore:

```bash
$ sudo umount /mnt/[SHARE]
```