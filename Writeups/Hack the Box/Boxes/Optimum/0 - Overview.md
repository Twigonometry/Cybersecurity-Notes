# Optimum Overview

This is the seventh box in my OSCP prep series.

**Box Details**

|IP|Operating System|User-Rated Difficulty|Date Started|Date User Completed|Date System Completed|
|---|---|---|---|---|
|10.10.10.8|Windows|3.4|2021-06-13|2021-06-13|

---

This box was a little more involved than some previous Windows boxes, and required a bit of playing around with exploits till I found a working one. Still, it was pretty simple. It just involved finding a code execution vulnerability in the HFS server that was running on the box. This got us a shell as the `kostas` user. From here we could exploit ms16-032 to get a shell as `SYSTEM`.

## Ratings

I rated the user stage a 2 for difficulty. It took me about 40 minutes, and most of that was finding the correct exploit out of a large number of potential ones. I rated the final part the same difficulty - finding the correct exploit was fairly simple, but my main issue was getting it onto the box and finding the syntax to execute it. This took me a couple of hours, which is slow going - but I'm getting faster.

Matrix Rating:

![[Pasted image 20210613134842.png]]

## Tags

#oscp-prep #no-metasploit #windows #web #hfs #kernel-exploit