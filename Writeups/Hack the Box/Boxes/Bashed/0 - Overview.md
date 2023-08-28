# Overview

This is the fifth box in my OSCP prep series.

**Box Details**

|IP|User-Rated Difficulty|OS|Date Started|Date Completed|
|---|---|---|---|---|
|10.10.10.68|3.3|Linux|2021-05-05|2021-05-05|

---

This box was pretty easy. The initial foothold was quite simple, and just involved digging around a website to find a webshell that had been left there by a developer. Getting user took me about half an hour including scans.

Priv esc took a little longer - I wasn't working on it with 100% concentration so missed a few key things that I should have spotted straight away. Specifically, `www-data` could run commands as `scriptmanager`, which allowed us to edit a script that was run by root and cause it to instead spawn us a shell. The priv esc was very simple once you found it, but I just looked in the wrong places for a little while. Overall the box took about 2.5 hours.

I did learn a bit about upgrading shells - part of the reason it took me a little longer was because working out of a webshell was slower due to its lower interactivity, but I found a nice new Python reverse shell that I can use in future.

## Ratings

I rated user a 2 for difficulty as there was a bit more guesswork and investigation involved to find a foothold, but ultimately the steps were simple and there was no privesc from `www-data` to `arrexel`.

I rated root a 2 also, but considered rating it a 3. I felt a bit rusty on this box, and struggled a little with remembering my best practices - but overall the privesc was very simple, and once I figured out where to look and managed to upgrade my shell it was plain sailing.

# Tags

#writeup #oscp-prep #linux #web #file-misconfiguration #no-metasploit