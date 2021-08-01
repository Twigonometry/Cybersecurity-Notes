# Shocker Overview

This is the eighth box in my OSCP prep series. It was also one of the boxes for Hack the Box's #takeiteasy dare challenge.

**Box Details**

|IP|OS|User-Rated Difficulty|Date Started|Date Completed|
|---|---|---|---|---|
|10.10.10.56|Linux|3.6|2021-06-14|2021-06-14|

---

This box had a straightforward expoit, but with some tricky enumeration. The initial foothold was provided via Shellshock, a common vulnerability in scripts that set environment variables. But due to a strange configuration in the server, finding the appropriate shellshockable file required you to use the exact combination of tools and wordlists, which took a while. Root access was more simple, and just involved a GTFOBin on perl.

## Ratings

I rated user a 3 - it took me about two hours to enumerate and find the Shellshock vulnerability, but only 20 minutes once I did. A lot of that time was documenting what I was doing - this is fine when I'm learning, but I need to be quicker on the exam.

I rated root a 2 - it was very simple, and only took me 20 minutes once I had a user shell.

Here is my matrix rating, which is quite high on all aspects:

![[Pasted image 20210614105519.png]]

Shellshock is widely exploited (or was for a long time), the foothold was just a CVE, and the box required a good amount of enumeration to find the vulnerable file. A good box overall!

## Tags

#oscp-prep #linux #no-metasploit #web #shellshock #perl #sudo #takeiteasy 