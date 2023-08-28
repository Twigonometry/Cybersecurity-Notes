# Website

## Basic Enum

We can see what powers the site by looking at its headers:

```bash
$ curl -v http://cereal.htb
*   Trying 10.10.10.217:80...
* Connected to cereal.htb (10.10.10.217) port 80 (#0)
> GET / HTTP/1.1
> Host: cereal.htb
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 307 Temporary Redirect
< Transfer-Encoding: chunked
< Location: https://cereal.htb/
< Server: Microsoft-IIS/10.0
< X-Rate-Limit-Limit: 5m
< X-Rate-Limit-Remaining: 149
< X-Rate-Limit-Reset: 2021-06-05T18:26:04.5220616Z
< X-Powered-By: Sugar
< Date: Sat, 05 Jun 2021 18:21:04 GMT
< 
* Connection #0 to host cereal.htb left intact
```

`Sugar` is an interesting addition in the `X-Powered-By` header that I've never seen before. There also appears to be a rate limit in place, which is something to bear in mind.

## Certificate

Immediately upon visiting the site, Firefox displays a warning about a self-signed certificate

![[Pasted image 20210406083553.png]]

Clicking 'View Certificate' reveals a subdomain, `source.cereal.htb`

![[Pasted image 20210406083753.png]]

(this was also present in the Nmap scan)

If we want to view the certificate again after accepting it, just click the padlock in the browser and the `>`, then `More Information`. This allows viewing the certificate:

![[Pasted image 20210605185025.png]]

## Login Form

Visiting the main site, we are just presented with a login form:

![[Pasted image 20210605184135.png]]

We can do some basic fuzzing of the form:
- try `admin:admin`
- try a simple SQL Injection with the username/password `' OR 1=1;--`
- try an SQLi polyglot to see if the for might be vulnerable: `SLEEP(1) /*’ or SLEEP(1) or’” or SLEEP(1) or “*/","password":"SLEEP(1) /*’ or SLEEP(1) or’” or SLEEP(1) or “*/"`

None of this gave any results. There is also seemingly no way to register - visiting `/register` gives us a blank page:

![[Pasted image 20210605184317.png]]

Let's take a look at the `source.cereal.htb` domain and see if there's anything else useful.