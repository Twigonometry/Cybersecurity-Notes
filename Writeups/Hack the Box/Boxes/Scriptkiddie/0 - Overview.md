# Overview

**Box Details**

|IP|OS|User-Rated Difficulty|Date Started|Date User Completed|Date System Completed|
|---|---|---|---|---|
|10.10.10.226|Linux|4.0|2021-02-13|2021-02-15|2021-02-16|

This was a pretty easy but really fun box based on exploiting another hacker's badly made website. The website runs a number of Linux commands in the background, one of which makes use of an outdated metasploit library. This can be used to execute commands on the box as the `kid` user by uploading a malicious APK file. From the kid user, we can escalate to the `pwn` user by exploiting a command injection vulnerability in a logging script designed to 'hack back' other hackers. Finally, root involves exploiting `sudo` permissions on the `msfconsole` binary to gain a shell.

---

I loved this box, and how meta it was. Every part of the path felt super on-theme and it was really enjoyable. It was one of those boxes where everything running on the box felt like it had a reason to be there, and wasn't just plopped onto it for the sake of having a CTF.

As with other writeups, this may contain screenshots dated from after the box retired. That's because I didn't use Obsidian when i did this box originally, and I went back and recaptured screenshots for this writeup.

I came back to this box to do the writeup after watching a couple of other videos and writeups on it, and found some nice alternative ways to do what I did originally. Revisiting the box really helped me analyse and be critical of my methodology the first time round, and I actually managed to pop a shell today using a certain method when I couldn't a few months ago.

## Ratings

I rated user a 3 for difficulty, and root a 4. The exploits weren't super complicated, just a little fiddly, especially when trying to return a shell. The initial APK exploit was really cool, and something I'd never heard of before. Enumeration was all pretty simple, and the final step to root was also easy. The meat of the box was the initial foothold and the escalation to `pwn`, so I'm sort of happy with the final step being simple.

## Tags

#writeup #web #cve #command-injection #linux