# Atom Overview

**Box Details**

|IP|Difficulty|OS|Date Started|Date User Owned|Date Completed|
|---|---|---|---|---|---|
|10.10.10.237|Medium|Windows|2021-04-19|2021-06-02|2021-06-22|

Atom was an interesting, but at times frustrating, box that involved pushing a malicious update file to an insecure Samba share, which exploited a CVE to get code execution on the box. Root involved finding some passwords in PortableKanban and Redis.

I liked the concept of this box, but it was super fiddly to execute and find the correct syntax. Situations like that frustrate me on Hack the Box, but I got there in the end. System involved a lot of poking around config files, which again isn't my favourite challenge, but it is an important skill to have and I learned how to interact with Redis, a common service.

I rated User a 4 for difficulty, as the exploit was a little obscure and the hints weren't explicit enough to not require extensive debugging (not that this is a bad thing, it just makes it harder). I rated root a 5 for difficulty, as it involved a lot of digging around and a few new tools I'd never used before.