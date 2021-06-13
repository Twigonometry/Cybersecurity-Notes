# Overview

**Box Details**

|IP|User-Rated Difficulty|OS|Date Started|Date User Completed|
|---|---|---|---|---|
|10.10.10.217|7.4|Windows|2020-03-16|2020-04-18|

This was a very hard box based on a .NET web application; it required a lot of source code analysis and involved a chain of exploits to get a foothold on the box. After reversing the authentication system and forging a JWT token, you could figure out how to insert malicious data into the database to trigger a deserialisation vulnerability. However, the triggering HTTP request had an IP address limitation, meaning only the box itself could trigger it. This meant we needed an XSS or SSRF vulnerability to force a request, which came via a CVE in a markdown rendering library. After this, we could trigger our deserialisation payload and upload a shell to the box.

A bit of advanced warning - this writeup is *long*. I have tried to include regular 'checkpoints' to let you skip ahead to working payloads. But a big part of this box for me *was* the debugging. It was intense, and full of little hurdles in the formatting of payloads and subtleties of scripts. I think it's important to include this so you can see what I learned. But I understand it's not for everyone.

---

I loved this box. It was my first dive into some really complex Web Application Hacking. I'd been looking at Deserialisation with my hacking society, but hadn't touched it in .NET, and the XSS on this box was really interesting too. It was great to chain everything together and find all the little pieces.

What also surprised me about this box was the sheer number of little hitches along the way. Maybe this was due to the way I approached it with having fairly little experience, but actually trying to debug things like only being able to communicate over HTTPS and sending data back to my box via `<img>` tags meant I learnt an awful lot while doing this. It was well worth the time investment, and truly was a lesson in persistence.

I only managed to finish the user stage of this box (although that was no small task). I had a look at the root path at the time, but due to my little Windows priv esc experience I decided to go back and focus on some retired boxes to try and learn a little more. I also had a CTF to run and an AWS exam, and by the time they were finished and I could come back to the box again it was about to retire. I might come back to the root stage at some point, but for now I'm uploading what I've got.

## Scripts

All the scripts I used on this box are available on my github, at https://github.com/Twigonometry/CTF-Tools/tree/master/hack_the_box/cereal

## Ratings

I rated the user flag a 7/10 for difficulty. It was pretty complex, and involved chaining several exploits together, each with little restrictions baked in just to screw you over.

# Tags

#writeup #web #xss #markdown #deserialisation #dotnet #windows