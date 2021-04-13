# Networking

## Get your IP

Use this command to see your IP address. `ifconfig` on its own will list all interfaces, or you can specify one.

```bash
ifconfig [interface]
```

specify `tun0` as the interface on HacktheBox

## Curl

Curl POST Request:

```bash
curl -X POST [options] [URL]
```

## SSH

### Generating a Keypair

Generate a public and private key with the following command. By default it will save to the `~/.ssh` directory.

```bash
$ ssh-keygen
```

You can also specify a key name with the `-f` flag. By default the private key will be called `id_rsa` and the public key will be called `id_rsa.pub`.

```bash
$ ssh-keygen -f key_name
```

### Secure Copy
```bash
$ scp [OPTION] [[user@]SRC_HOST:]file1 [[user@]DEST_HOST:]file2
```