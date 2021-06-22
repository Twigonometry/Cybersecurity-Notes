# Overview

This is the fourth box in my OSCP prep series.

**Box Details**

|IP|User-Rated Difficulty|OS|Date Started|Date Completed|
|---|---|---|---|---|
|10.10.10.95|2.9|Windows|2021-05-04|2021-05-04|

---

This box was pretty easy. It involved logging into a Tomcat Manager page and uploading a `.WAR` shell, which gave us `system` access.

It took me about two hours, which is pretty slow compared to the six minutes for first blood. However, that was mostly because I spent a fair amount of time down a rabbit hole trying to exploit a CVE. When I found the correct path through the management console, it took me about half an hour; the exploit itself dropped me in directly as `system`.

## Ratings

I rated both user and root a 1 for difficulty. The exploit was arguably even simpler than Blue and Legacy, at least by hand - while it wasn't a case of just firing off a metasploit module, uploading a shell and triggering it by simply visiting the URL is much simpler than manually editing an exploit. There was also no privesc involved.

# Tags

#writeup #oscp-prep #windows #file-upload #tomcat #no-metasploit