# Overview

This is the third box in my OSCP prep series.

**Box Details**

|IP|User-Rated Difficulty|OS|Date Started|Date Completed|
|---|---|---|---|---|
|10.10.10.3|2.6|Linux|2021-05-03|2021-05-03|

---

This box was also very simple, again only made harder by manual exploitation. It involved exploiting a CVE in SMB that allows command injection via the username field.

Unlike Legacy and Blue there were a couple of different services to look at. As I looked at FTP first, it took me a little longer to root as I spent some time trying to get that exploit working. Overall the box took me about 2 hours, but I learnt two exploits along the way.

## Ratings

I rated both user and root a 2 for difficulty, as there was a little more to them than just firing off metasploit at a single service, and the manual exploitation had a few different paths.

# Tags

#writeup #oscp-prep #unix #cve #smb #ftp #no-metasploit