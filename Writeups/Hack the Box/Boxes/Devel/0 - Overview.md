# Devel Overview

This is the ninth box in my OSCP prep series.

**Box Details**

|IP|OS|User-Rated Difficulty|Date Started|Date Completed|
|---|---|---|---|---|
|10.10.10.5|Windows (7)|3.6|2021-06-14|2021-06-14|

This was a great easy box that involved uploading a webshell to a site via an FTP directory that linked to the webroot. Privesc to system involved exploiting `SeImpersonatePrivilege` using Juicy Potato. The box took me just over 90 minutes in total, which is a nice time frame for a low-end OSCP box equivalent.

---

This was my first time successfully running a potato exploit, and it feels good to have another tool under my belt!

I can also feel myself getting better at windows - I'm faster, and more familiar with the tools, tricks, and file upload methods. This is all pretty basic stuff still, but I'm enjoying laying the foundations.

I think although I'm getting faster, I need to be quicker taking notes. This process definitely slows me down, and I'm sure I can cut out a lot of the markdown formatting until the end when the box is finished.

## Ratings

I rated both flags a 3 - foothold was extremely easy, and I would have rated it a 1, but I couldn't read either of the flags until I got SYSTEM on the box. The priv esc was slightly harder - Juicy Potato is very well documented, but getting all the steps together to execute it wasn't straightforward enough to rate it lower.

Matrix Rating:

![[Pasted image 20210614132628.png]]

## Tags

#oscp-prep #windows #potato #no-metasploit #web #asp #ftp