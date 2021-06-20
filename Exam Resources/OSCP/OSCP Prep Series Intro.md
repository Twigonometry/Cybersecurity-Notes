# OSCP Prep Series Intro

As I prepare for my OSCP exam, I will be blogging about all the things I've learned. This will mostly be in the form of Hack the Box retired machine writeups, but I'll also be regularly adding to my Cybersecurity Notes repository with the things I've learned. This might come in the form of detailed posts about topics, such as a Buffer Overflow or Web Hacking methodology guide, or a cheatsheet such as a list of useful Windows commands I've used along the way.

After I've sat (and hopefully passed) I'll do a big roundup of what I learned and how I found it... even if I fail. I imagine this will still be useful even if I flunk the exam!

I hope to sit the exam in late August 2021. I will be booking 30 days of lab time, starting in July, and will then have a month-ish to sure up my skills before sitting.

Until then, I will be posting regularly here! I hope you enjoy the writeups and find the style helpful. Below are some of my thoughts on what I'm including in them.

## Writeup Style & Methodology

My writeup style might not fit everyone's tastes - I tend to include a lot of detail about my thought process and things I did to debug when stuff goes wrong. While this makes the writeups a lot longer, it is helpful for me in the long run so I can see what I did wrong and how I fixed it.

What I want to include in my writeups:
- My thought process while solving the boxes - i.e. what I tried and why I tried it
- My mistakes - i.e. what went wrong, and why?
- Key learnings for each box - new technologies, command line tricks, exploits and CVEs, and defence bypass techniques I used
- Setup instructions when using technologies and tools for the first time - this way, I can reference this section in a later writeup

If that sounds good, then these writeups might be beneficial to you! Still, I will include an Index page (in [Obsidian](https://github.com/Twigonometry/Cybersecurity-Notes)) or a Contents section (on [my site](https://mac-goodwin.com/blog/htb/)) to help with navigation, and regular checkpoints so you can skip ahead to what worked.

With all that said, my first post in the series is [[Blue Index|Blue]]. I will be adding tags to my blog page in the future so that the OSCP boxes are easier to find - they will be marked with the #oscp-prep tag.

## What I'm Doing to Practice

### Course Content

Of course, I still need to actually *do* the course. Right now, I'm just muddling through Hack the Box. I will book the course to start in July, do my 30 days of lab time, and then I'll hopefully have a couple of weeks to revise before sitting the exam. 

I was advised to do a decent amount of Hack the Box so that by the time I sit the course I should be able to breeze through the labs within the 30 days. We'll see how that goes...

Either way, I am hoping doing the course and the labs will help me have a better understanding of where I'm at - right now it is hard to tell whether I'm ready or not, especially as the style of Hack the Box machines might not match up exactly to the style of machines on the exam. I'm hoping the labs will be more representative and show me where my weaknesses are and what the top-end of difficulty is so I have time to polish off my skills before I sit the exam.

### Hack the Box

Here are the boxes I'll be prioritising, sorted by user-rated difficulty - I'm about halfway through as of writing this:

- [x] Blue (Windows - 2.4)
- [x] Legacy (Windows - 2.5)
- [x] Lame (Linux - 2.8)
- [x] Jerry (Windows - 2.9)
- [x] Bashed (Linux - 3.3)
- [x] Granny (Windows - 3.3)
- [x] Optimum (Windows - 3.4)
- [x] Shocker (Linux - 3.6)
- [x] Devel (Windows - 3.6)
- [x] Sense (Linux - 3.6)
- [x] Nibbles (Linux - 3.7)
- [x] Beep (Linux - 3.7)
- [x] Poison (Linux - 4.0)
- [ ] Valentine (Linux - 4.1)
- [ ] Arctic (Windows - 4.1)
- [ ] Sunday (Linux - 4.2)
- [ ] Solidstate (Linux - 4.2)
- [ ] Bounty (Windows - 4.6)
- [ ] Silo (Windows - 5.2)
- [ ] Nineveh (Linux - 5.3)
- [ ] Tartarsauce (Linux - 5.8) 
- [ ] Node (Linux - 6.0)
- [ ] Brainfuck (Linux - 6.6)

This list is comprised mostly of those from the [TJ Null List](https://www.reddit.com/r/oscp/comments/cu6jhb/updated_oscplike_boxes_from_hackthebox_by_tjnull/), up to a difficulty level that has been described as probably the top-end of OSCP boxes by colleagues who've sat the exam.

I'll be saving these boxes for a practice exam, as per a [reddit recommendation](https://www.reddit.com/r/oscp/comments/e3omhp/htb_practice_what_would_count_as_a_10_pt_box_20/), which I will writeup on this blog:

- [ ] Grandpa (Windows - 3.5) (10 points)
- [ ] Cronos (Linux - 4.2) (20 points)
- [ ] Bastard (Windows - 4.5) (20 points)
- [ ] Kotarak (Linux - 6.5) (25 points)

I'll also add the Vulnhub machine Brainpan to this list as the 25-point BOF box.

I won't write up the real exam (because I can't) but hopefully a post about how I found a practice exam will help some people, especially with explaining relative difficulty to Hack the Box and showing techniques for time management.

### Other Practice

Besides these boxes, I'm planning a few other activities to help me prepare:
- Compiling all my notes from my practice into cheatsheets
- Extra Buffer Overflow Practice
	- Reading [Corelan's Buffer Overflow Series](https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/)
	- Watching [Live Overflow's Series](https://www.youtube.com/playlist?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN)
	- Doing Jack's [Stack Overflows for Beginners](https://www.vulnhub.com/entry/stack-overflows-for-beginners-101,290/) machine
- Practicing Privilege Escalation
	- Tiberius' [Linux Priv Esc THM Room](https://tryhackme.com/room/linuxprivesc)
	- Tiberius' [Windows Priv Esc THM Room](https://tryhackme.com/room/windows10privesc)

If I have time, I'll do the following
- The rest of [TJ Null's list](https://www.reddit.com/r/oscp/comments/cu6jhb/updated_oscplike_boxes_from_hackthebox_by_tjnull/), including the harder boxes such as Jail which I missed off my initial checklist
- Other boxes from 0xdf's [OSCP-like](https://0xdf.gitlab.io/tags.html#oscp-like) and [OSCP-plus](https://0xdf.gitlab.io/tags.html#oscp-plus) lists
- Heath Adams' [Windows Priv Esc Course](https://www.udemy.com/course/windows-privilege-escalation-for-beginners/) and [Linux Priv Esc Course](https://www.udemy.com/course/linux-privilege-escalation-for-beginners/), or Tiberius' [Windows](https://www.udemy.com/course/windows-privilege-escalation/) and [Linux](https://www.udemy.com/course/linux-privilege-escalation/) equivalents
- Try Hack Me rooms about [Buffer Overflow](https://tryhackme.com/room/bufferoverflowprep), [Active Directory](https://tryhackme.com/room/activedirectorybasics), and [Kerberos](https://tryhackme.com/room/attackingkerberos)
- More modern HTB boxes, such as the recent releases Monitors, Dynstr, and Spider

## Progress So Far

I've learned a lot from these boxes so far. One of the main things that has improved is my methodology - my enumeration steps feel pretty solid now, and I know how to interact with a lot more core services. I'm also much better at privilege escalation than I was, and have started to learn where to look and what stands out as unusual on a filesystem (as well as learning about various dangerous configurations like capabilities).

Crucially, I'm much better at windows. Not only am I more comfortable with Command Prompt and Powershell, but I've had the chance to explore various kernel exploits and techniques for finding them, as well as successfully exploiting a Potato-family exploit for the first time and understanding *why* they work. I think I'd now be confident to go back and try a harder example of this, such as Cereal's manual exploitation.

On top of this, I've learnt a lot about file transfer on Windows systems, generating shellcode with `msfvenom` for various languages, and troubleshooting shells on several Operating Systems. I've also had some experience modifying and searching for CVE PoCs, which is a crucial skill.

Finally, I've come across a few nice classes of common vulnerabilities and attack vectors, such as log poisoning and shellshock, which I just wouldn't have known to look for before taking on the retired boxes.

## What I still Need to Achieve

### A Rock-Solid Methodology

I feel like my methodology has massively improved over the last six months or so of practice on HTB (and with [[SESH Index|SESH]] and other side projects), but it's still worth being critical of it. Here's what I think I need to improve:
- Not overlooking underlying webserver technologies - on a few boxes, such as [[Granny Index|Granny]], I initially overlooked what was running on the webserver (i.e. WebDAV) as not important and focused on the powering language like ASP. Being meticulous about what shows up in the initial `nmap` scans will be important going forward
- Realising when I'm in a rabbit hole - on a few of these boxes I've spent a long time debugging an exploit before realising it's not going to work. Sometimes it is hard to tell that it isn't appropriate for the box, such as on Granny when I only realised Juicy Potato wouldn't work when I got to the point of supplying a CLSID, but I think going forward I should set myself a cutoff point when I'm stuck where I force myself to try something else
- At the same time, I need to not give up on an exploit too quickly - a few times I've searched for a class of exploit and not got a result immediately, and moved on to the next. It's worth googling for alternative implementations of that exploit, or adjusting my payload until my options are exhausted
- Trying every exploit - in a similar vein, I've often overlooked `searchsploit` results, or forgotten to try them after getting deep into one that didn't work. A checklist might help going forward so I can refer back to my possible attack vectors when I get stuck

### Faster Note Taking

Note taking bogs me down. There's no denying it. I think this is because I'm simultaneously trying to create a writeup while I solve the box, so that I can quickly publish it. In reality I always spend a lot of time tweaking my writeups afterwards, so writing out my thought processes and narrating what I'm doing probably doesn't save me much time.

Writing out my thought process does *sometimes* help with solving the box - however, I need to make it snappier and focus on key details, then flesh it out afterwards.

This shouldn't be a problem on the exam itself, as I won't be publishing my exam solutions anywhere (obviously). But if I want to work through retired boxes efficiently, I need to learn how to take notes with a balance between speed and detail.

### General Speed

I spend a lot of time comprehending what I'm about to do. I think this stems from an innate fear of doing something that will screw everything up - I'm usually hesitant to make a big change on a computer system, like migrating a database, editing a partition, or running my code for the first time.

I don't think this apprehension is unreasonable - but sometimes I need to just bite the bullet and launch an exploit for the sake of speed, rather than staring at the output of `searchsploit`.

I think recently a lot of this has come from burnout, too - I will often find myself staring at a box if I've done a few that day and just not being able to push forward or feeling like the attack path seems too complicated. When I get this feeling, it's a good sign to take a break - and it will be crucial to recognise on a 24-hour exam.

### Avoiding Tip Addiction

With retired boxes, it is *very* easy to go online and look up the answer. I often find myself validating that I have the right approach, or checking my payload against a writeup.

There's nothing inherently wrong with taking hints - it's all learning, after all. But it needs to be consistently done as a last resort, not just when I'm a bit stuck. I'm making a commitment to exhaust all my ideas and avenues of attack before checking online from this point forward.

### Windows

While I've gotten better at Windows, I still feel I have a few key areas that I need to brush up on:
- Compiling exploits locally
- Interacting with the registry
- DLL injection-based attacks
- Understanding windows security features such as the storing of credentials and how privileges work
- Different constrained language modes and how to escape from them
- Key techniques to exploit different kinds of limited code execution during priv esc, such as adding a new user - this would be in a situation where an exploit didn't just give me a shell, and I needed to do something more complicated

### BOF

Buffer Overflows are a big daunting thing to learn for me - I am hoping that the OSCP course content will sure up my understanding, and I know I need to practice the practical side of exploitation with things like `gdb` so I know how to apply the theory.

### Other

- Non Web-Based Challenges and Services - I have to say web is my favourite part of cybersecurity so far, but need to not neglect things like Active Directory
- Docker and other container-based services seem to be becoming more popular on challenges recently, and I'd like to get more familiar with enumerating and escaping them
- Advanced SQL Injection - I need to brush up on practical exploitation of harder SQL challenges, such as blind injections, and ways to pop shells from SQL environments. It's also worth looking at NoSQL technologies too, like MongoDB
- Proxies and advanced Networking Concepts - practically using proxies and tunnelling technologies like `socat` and `proxychains` is a skill I'm not as comfortable with as I'd like. I also want to look at interacting with services like Squid, which I've seen come up on a few more recent HTB machines

## Conclusion

I'm excited to continue with my OSCP journey, and I hope to smash the exam! Here's hoping that these collected resources will be useful to somebody sitting the exam in the future - from one novice hacker to another.