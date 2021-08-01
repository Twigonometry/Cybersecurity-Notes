# Key Lessons Learned

- FTP can upload in binary mode (`ftp> binary`) which is useful when pushing `.exe` files
- Use of Juicy Potato to escalate to SYSTEM, using a powershell reverse shell
- Read `systeminfo` carefully - you might see an `x64` processor, but the system may actually be 32-bit