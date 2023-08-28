# Website

## Login Form

Visiting the homepage presents us with a login screen:

![[Pasted image 20210330103246.png]]

There is a 'powered by' string in the footer, but no obvious version number. The software listed is `Arnageddon`, a slight variation on spelling that is easy to miss. On previous boxes like Doctor this one letter off has been relevant, but after a quick check on searchsploit to see if there are any documented vulnerabilities it seems it was just a misspelling:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/armageddon]
└─$ searchsploit arnageddon
Exploits: No Results
Shellcodes: No Results
```

A quick check of the source confirms that Drupal 7 is used:

![[Pasted image 20210330104241.png]]

I'll come back to this, but do some manual poking of the site first.

It seems we can register for an account. Let's do this, passing the request through Burp.

![[Pasted image 20210330111528.png]]

There don't seem to be any useful hidden fields that could be edited to give us extra permissions.

We get this message when we submit. If we had submitted a password when we created the account then I might attempt logging in despite the account needing approval, but it looks like we are stuck for now.

![[Pasted image 20210330111117.png]]

I tried some basic SQL Injections on the login form, submitting `' OR 1=1;--` in the username and password fields. I also tried the polyglot `SLEEP(1) /*’ or SLEEP(1) or’” or SLEEP(1) or “*/` from [this article](https://dev.to/didymus/xss-and-sqli-polyglot-payloads-4hb4). However, it did not trigger any errors.

## Drupal Directory Listing

I wasn't sure where to go from here, so started to look at the results from the Gobuster scan.

Being unfamiliar with Drupal, I decided to start with `/profiles` in case it had any details of usernames registered on the platform. This revealed a directory listing:

![[Pasted image 20210330113044.png]]

Lots of these files are from 2017, which suggests the version of Drupal that is running is an old version. Running `searchploit drupal` reveals many results concerning Drupal 7. Let's see if we can use one to get a shell.