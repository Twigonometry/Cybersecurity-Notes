# Overview

This was the second box I did from the Hack the Box Take it Easy Dare Challenge.

**Box Details**

|IP|OS|User-Rated Difficulty|Date Started|Date Completed|
|---|---|---|---|---|
|10.10.10.138|Linux|4.4|2021-07-15|2021-07-15|

This box started with a bit of digging around a blog for something exploitable - unfortunately there was a WAF (Web Application Firewall) preventing brute forcing and fuzzing, so it was back to basics. Eventually I found a version number for a CMS which had an SQL Injection vulnerability, allowing us to extract a password hash and log in to the box.

The privesc to root involved exploiting a Message of the Day script that called a binary without an absolute path - the `/usr/local/sbin` directory on the box was writeable, which meant we could hijack the binary by writing a file with the same name at that location on the path.

## Ratings

I rated the user flag a 2 for difficulty. I spent a long time enumerating it, but realistically I would have found it much easier if I'd paid a bit more attention to the source code. Once I spotted the framework, I'd shelled it within 20 minutes. The box was made a little harder to fuzz by the WAF, but scanning wasn't actually necessary for the user flag.

Root was slightly trickier, and involved a bit of SUID trickery to get a shell. It was also a little tricky to find the target binary and the right syntax, but the exploit concept wasn't too hard. It took 45 minutes to read the flag itself, then another 10-15 minutes to figure out how to get a root shell.

## Tags

#linux #no-metasploit #web #cve #cron #pspy #motd #takeiteasy 