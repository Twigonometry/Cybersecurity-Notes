# Overview

This is the sixth box in my OSCP prep series.

**Box Details**

|IP|User-Rated Difficulty|OS|Date Started|Date Completed|
|---|---|---|---|---|
|10.10.10.15|3.3|Windows|2021-06-11|2021-06-11|

This box was a little frustrating for me, but highlights some good fundamental Windows skills; file upload, ASPX webshells, enumeration via `systeminfo` and Windows Exploit Suggester, and token kidnapping exploits as an alternative to the potato family of exploits.

The foothold lay in a file upload via WebDAV - with a bit of trickery, you could upload and trigger an `.aspx` shell. This gave you access to the box as a Network Service; from here, the account's special `SeImpersonatePrivilege` privilege could be used to kidnap a token for the SYSTEM account and spawn a shell as SYSTEM.

---

I originally started this box on the [SESH Account](https://www.hackthebox.eu/profile/451740) on 2021-05-06. I spent a good few hours getting a shell as Network Service, and got stuck from there. I tried a lot of different exploits, and eventually set the box aside. I came back to doing retired boxes after the SESH CTF and sitting my AWS CCP exam with a clearer head and gave it another go from the start.

I'm not going to lie - I stuggled with this box. Sometimes it's demoralising (and worrying) to spend multiple hours on one of the easiest HTB boxes. But while my methodology could improve (and is improving) I'm happy with my persistence - I exhausted pretty much every avenue I thought of before landing on the correct exploit, and I'm glad I did.

By far the most difficult part for me was finding the correct exploit for SYSTEM access - I feel like it was difficult to find without googling something quite specific, which just highlights the value of knowing *how* to search effectively. Frustratingly, after reading writeups I realised my tools did not highlight the exploit where they did for other people running similar commands. This is not a great sign for the exam, knowing that sometimes enumeration scripts may behave inconsistently, but with a little persistence you can always get there.

___

I feel like I am slowly becoming more familiar with Windows exploitation, and its various classes of privilege escalation - so far I have mostly seen kernel exploits, and the boxes I've done since Granny have involved the potato family of exploits (which I'll [[15 - Shell as Network Service#Trying Juicy Potato|touch on]] here). I've also picked up a good few skills with file transfer, building reverse shells, and enumerating the operating system. Windows boxes before this one had mostly dropped us into the box as SYSTEM, so I'm glad to have some meatier challenges.

## Tags

#writeup #oscp-prep #no-metasploit #webdav #web #asp #seimpersonate #ms14070

## Ratings

I rated both user and root a 3 for difficulty. The concept of the foothold wasn't super complicated, but had a couple of tricky syntax requirements and required some creative thinking with renaming files. From the network service account, there seemed like a large number of attack vectors - while the correct exploit was simple enough to execute, it was difficult to find amongst all the alternatives.