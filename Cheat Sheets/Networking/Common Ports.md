# Common Ports

## What is a Port?

A port is simply a channel over which a computer can communicate. Think of it like having two letterboxes - one for large parcels, and one for letters.

Ports can send and receive data. A computer can open and close ports at any time, and many services will communicate over a specific port by default. It's like going to the post office and knowing which desk you should go to to send a parcel or a letter.

The reason a computer has multiple ports is so different services (like email, SSH, and HTTP traffic) can be directed to specific parts of a computer and avoid congestion.

## Common Ports

Lots of common services communicate on a specific port by default. If you see these ports come up in a scan, you know what they might do.

### 22 - SSH

Port 22 usually hosts SSH. This stands for 'secure shell' and is a way of remotely accessing a computer.

If you see port 22 open, it means you may be able to remotely access the computer if you have a user's SSH password or SSH private key.

See more:
- [[Fundamental Skills#SSH]]
- [[CVE-2012-5975]]

### 80 - HTTP

Port 80 serves HTTP traffic. This is the default port for unprotected traffic sent over the web - for example, a `GET` request to visit `http://www.google.com`, or a request back to your own webserver that is hosting an enumeration script.

If you see HTTP open, it means there should be a publicly accessible website - always a good starting point.

See more:
- [[Fundamental Skills#Sending HTTP Requests]]

### 443 - HTTPS

Port 443 serves HTTPS traffic, which is just like HTTP but encrypted using TLS.

This is essentially a 'secure' version of HTTP, and is used to protect sensitive data being transmitted over the internet.

If you see this port open, there is likely a HTTPS version of the website you are visiting. It may mean you will be able to view the site's SSL/TLS certificate, which can sometimes contain useful information (including domain names).