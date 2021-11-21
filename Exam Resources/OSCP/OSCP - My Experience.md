# My OSCP Experience

On 10th October 2021 10:00 I sat my OSCP exam.

This was after months of preparation (and impostor syndrome). Alongside 90 days of lab time I did plenty of Hack the Box, completed a Year in Industry placement, and finished my first year of running my University's [Cybersecurity society](https://shefesh.com/).

All of these things helped massively with my exam preparation, and doing my OSCP has achieved the same effect in reverse; I'm now much more confident giving lectures on Cybersecurity, mostly thanks to the experience gained from this course, and I'm looking forward to seeing how it can be applied to whatever job I end up doing.

The exam had one of the worst starts I could have imagined; it took about 20 minutes to connect to the proctor, I couldn't scan my ID, and the exam ended up starting about 25 minutes late. Despite this, OffSec support were fantastic and the rest of the exam ran pretty smoothly. I'll talk more about the actual exam later, but I eventually completed 70 points worth of boxes and finished, in theory, with a pass.

I was pretty drained, so I wrote the report pretty slowly over the next 24 hours. I submitted it at roughly 09:25 on the 12th October, and heard back by 09:35 the next day. This was such an unexpectedly quick response that I didn't even think to check my emails until 18:30 that day, which is when I found out I'd passed!

Doing the OSCP was a gruelling experience, with lots of low points (and a lot of money spent). But I learnt an awful lot, and I'm extremely grateful to my work colleagues for giving me the time to pursue it and to my friends in the industry for all their advice.

I will say that it's a very unforgiving certification for anyone with any other major responsibilities (work, University etc). It's possible (clearly), but I had to spent a lot of my spare time working towards it. There's a lot more that I wanted to do to prepare, and would have done if I had more time - I'll talk more about this [[#Things I Wish I'd Done|later]] in the blog post.

I hope this blog (which I know is just one of many rambling OSCP journey posts) will be interesting and helpful to anyone looking to do the certification. It helps to know that someone with a similar background to you can pass the certification, so I hope this will inspire some confidence in at least *one* OSCP student. In each section of this blog I'll outline what I think is the most important lesson I learned.

If you'd rather not read more, I'll be adding a lot of resources to this repository whenever I get time, including cheatsheets, notes on my methodology, and key lessons learned. But if you want a deeper dive, read on to find out more about my background, how I prepared, and my strategies during the exam!

You can also find this note in the form of a blog post on [my website](https://www.mac-goodwin.com/blog/cyber/2021/10/16/oscp-experience.html).

## Contents

- [[#My Background]]
- [[#Pre-Course Preparation]]
  - [[#Getting Serious]]
- [[#The Course]]
  - [[#Practice Exams]]
  - [[#Tips and Feedback]]
  - [[#Attacking the Labs]]
- [[#The Exam]]
  - [[#The Report]]
  - [[#Exam Tips]]
- [[#Things I Wish I'd Done]]
- [[#What Next]]

## My Background

> You don't need to be a natural hacker to do this cert - you just need the willingness to learn (and a lot of spare time)

I've always had an interest in computers, and in Computer Science since I had my first programming lesson in 2013. I'm currently doing a Master's Degree in Computer Science at the [University of Sheffield](https://www.sheffield.ac.uk/dcs) and have just finished a public-sector Year in Industry placement.

This may be impostor syndrome speaking, but I don't feel like a 'natural' at Cybersecurity; I wasn't one of those kids who took apart and reassembled calculators, and I have to work pretty hard to understand a lot of the concepts that seem to be intuitive to many researchers in the industry. But I have the drive to learn, and that's really all you need to pass this certification.

I've been interested in Cybersecurity for a while, but only started taking it seriously as a career option when I started University in 2018. I joined the [Sheffield Ethical Hackers Society](https://shefesh.com) (SESH) as a member, panicked that I didn't understand anything, and didn't really attend much in the first year. After a bit of work experience at NCC Group and with my first year of Uni under my belt, I came back to the society in 2019 and started attending more regularly.

The society is where I found out about my future Year in Industry employer, and where I learned a lot of key skills. I started trying Hack the Box and participating in the odd CTF around this time, but still found hacking pretty difficult and didn't practice regularly enough to make much progress.

SESH is also where I first heard about the OSCP, and I immediately thought it would be a great thing to aspire to. I started thinking about it more seriously once I started my placement, and began obsessively reading 'OSCP Journey' writeups like this one on [Reddit](https://reddit.com/r/oscp) to try and figure out when I might be ready to sit the exam.

## Pre-Course Preparation

> Don't start the course too early - make sure you have a good grasp of important concepts such as Computer Networking, Command Line interfaces, and maybe even basic Cryptography before you start

I didn't feel confident enough to start the material until about April this year, which is when I started drafting up how I might approach it. My first tip for taking the certification is to get a good sense of how long it might take you; I wanted to do it before my Year in Industry placement finished (which didn't happen) and knew that I'd want to do a significant amount of preparation before I started the course.

I started doing Hack the Box machines more regularly, watched a lot of InfoSec content and read [a lot](https://ranakhalil101.medium.com/) of [writeups](https://0xdf.gitlab.io/) to improve my methodology, and even built a CTF with SESH. I also gained a lot of experience with secure development and networking via my Year in Industry, which definitely helped indirectly. I'm glad I didn't start the actual course too early; the skills I gained from watching [IppSec](https://www.youtube.com/channel/UCa6eh7gCkpPo5XXUDfygQQA), doing Hack the Box, and running SESH were invaluable.

If you're completely new to Cybersecurity, I thoroughly recommend getting comfortable with key concepts such as Networking and Command-Line interfaces, or practicing basic penetration testing methodologies on a platform like TryHackMe, before you start the course. While some of these concepts are taught on the course, it will mean you spend less time grappling with the basics and get more time attacking the labs.

Many writeups even recommend doing an intermediate certification before starting the OSCP - while I haven't done any of these myself, the CompTIA [Security+](https://www.comptia.org/certifications/security) and [Network+](https://www.comptia.org/certifications/network) courses are highly recommended, and OffSec have even released a new Fundamentals Course (PEN-100) available through their [Learn](https://www.offensive-security.com/learn/) programme. While I'm promoting beginner resources, we are currently producing a [Fundamental Skills](https://shefesh.com/wiki/fundamental-skills/) series at SESH which teaches a lot of these key concepts and is available for **free!**

### Getting Serious

I decided I'd start the course in June, and hopefully sit the exam at the end of August. In the month before buying the course I ramped up my preparation on Hack the Box, and was lucky enough to get some allowance to do this during work hours. Without this, I wouldn't have been able to pass the exam and I'm extremely grateful to my amazing Year in Industry team :)

I spent a few weeks doing this, working roughly through the [TJ Null list](https://www.netsecfocus.com/oscp/2019/03/29/The_Journey_to_Try_Harder-_TJNulls_Preparation_Guide_for_PWK_OSCP.html#vulnerable-machines) of HTB boxes and making detailed [writeups](https://www.mac-goodwin.com/blog/htb/) of each. This was an invaluable process, and helped me learn to take good notes in preparation for the lab environment and the exam. Not only did these notes help me keep track of key details when solving the boxes, but they also were a brilliant reference guide when revising. You can find many examples of how I laid these out in this repository.

I also made myself a rough timeline of how I expected my preparation to go, and when to start each task. This was valuable in organising the *massive* list of potential things you can do to prepare for this certification, but the timeline changed a lot as my preparation went on.

## The Course

> Don't arbitrarily restrict yourself to sitting the exam at a certain date; sit it when you truly feel ready, and give yourself enough lab time to facilitate this (also, the best way to test whether you're ready is a practice exam!)

I bought the course in June, and made my first big mistake. I bought only 30 days of lab time, mostly due to my self-imposed restraint of sitting the exam in late August. The plan was to do as many labs as I could in a month, then spend July consolidating my knowledge and doing some final practice.

If you don't know, there's a 7 day delay between buying the course and being sent the VPN credentials and course material. I spent this week doing more HTB, and when my course material arrived I dived straight into it.

This was my second mistake, although not a huge one; I spent probably three days reading the course material before starting the labs. While the course material is fantastically written (and especially good if you're a complete beginner) I don't think it is necessarily worth as much of your time as the labs are. Here's a few reasons why:
- The labs are, at the end of the day, the best test of the skills needed for the exam. You need to know how to compromise a machine from just an IP address, and while the course teaches the skills you need to do this in isolated sections it doesn't give the best overview of how to chain the steps together
- Doing the frankly ridiculous amount of course exercises isn't worth the 5 extra points it will gain you by submitting a lab report; the odd exercise is fine to test a skill you're uncomfortable with, but you'll learn the skills more effectively by using them in a real application (i.e. a lab machine)
- The course materials can be used as a reference guide when performing specific exploits on the labs; I regularly referenced sections of the PDF I hadn't actually read to check OffSec's advice on a certain attack vector or privilege escalation method
- You can always read the material *after* your labs end; you have 120 days to schedule your exam within, which means there's time to revise before you sit it

If you take my advice from the [[#Pre-Course Preparation|Pre-Course Preparation]] section and revise these basic concepts ahead of purchasing the course, you'll be able to skip a lot of the initial reading and jump straight into the labs. I can't recommend having a good fundamental grasp of security before starting the OSCP enough. It is *not* an easy course to complete in a month with no prior knowledge.

Back to the labs; as it does, life got in the way of my precious 30 days, and by the end of my lab time I'd only rooted five boxes. Work were kind enough to put some money towards a lab extension, and I heavily prioritised my revision for the next month and managed to root a good number of machines. I was feeling confident, and scheduled my exam for the 6th September.

By the time my extension was about to end, and the exam was just over the horizon, I had 19 boxes under my belt. I wasn't sure whether this was enough, and knew there were still plenty of specific lab machines and extra prep tasks on my list. I also had a holiday in late August, and despite my best intentions to revise while I was away I did no such thing.

I ended up reading this [excellent post](https://www.offensive-security.com/offsec/pwk-labs-success/) by OffSec that had some scary statistics about pass rates. Reading this just confirmed my suspicion that I wasn't ready, and when I was back from holiday I relucantly took my wallet out and purchased another 30 days. I also rescheduled my exam, this time for the 10th October.

This extra lab time turned out to be crucial, and I feel like I learned more in this 30 days from the labs than before; this might be because I had just finished work and could fully focus on them, or because I knew it was realistically my last shot at revision. I wanted to get the most I could out of the labs, and experience as many different attack vectors as I possible (even if they weren't as relevant to the exam). In the end, I completed 33 lab machines including two network pivots, two mini Active Directory networks, and two of the 'Big Three' machines (Pain and Sufferance). I learned lots of new techniques, including plenty of kernel exploits, client-side attack vectors, and how to use [BeEF](https://beefproject.com/).

The key lesson here is this - **if you don't feel ready, you're not.**

In the end, I completed:
- 33 lab machines
- 46 Hack the Box machines
- 5 VulnHub machines
- Two practice exams (detailed below)
- Countless hours of research for Sheffield Ethical Student Hackers presentations

By the time my real exam rolled around I felt *much* more confident, and felt like I had made the most of the labs and course material. There was still more I could have done, but I'd worked hard enough to feel like I could pass.

### Practice Exams

Besides the lab time, I did two practice exams. The practice exams were **the most valuable preparation I did** for the OSCP. Here's why:
- You will know what it feels like to be working on an exam for 24 hours straight, and how to take care of yourself (food, water, breaks, sleep) on the day 
- You'll learn how to write a report, and crucially what information you *wish* you had recorded during your practice; writing a report and realising I was missing screenshots from when I gained a shell, for example, taught me what I needed to capture on the day
- You'll know what start time works for you; if you do a practice exam that starts in the morning and one that starts in the afternoon, you can pick a time that worked better for the real exam
- You'll learn a strategy for what order to tackle the machines, how to escape rabbit holes, and what to do when you're stuck and **cannot** take a hint

If you want to read more about my practice exams, including the reports I produced, you can do so [[OSCP Practice Exam Writeups|here]].

### Tips and Feedback

I really enjoyed the course - the lab environment was incredible, and the material brilliantly written and easy to follow. However, I wish it was more friendly to people with less spare time. I think if OffSec offered a grace period and made the course material available to students in the week before the labs start it would make the experience far less stressful, and would allow complete beginners to dive straight in to the course material without feeling like they're wasting lab time.

I would also recommend booking your exam plenty of time in advance - this is sometimes difficult if you're not sure whether you will be ready (it's very difficult to know this), but remember **you can book and then reschedule your exam**. I would look at dates a few weeks in advance of when you wish to ideally sit the exam (especially if you want a weekend) and then reschedule if you don't feel ready in time.

I also suggest buying at least 60 days of lab time from the start. It's cheaper to buy extra time at the time of signing up for the course than it is to purchase an extension; if you feel extremely confident in your abilities as a penetration tester you may be able to complete all the labs in a month, but for most people this isn't feasible. Having 60 days or more will also let you get a sense of the difficulty before booking your exam.

### Attacking the Labs

Here's my advice for attacking the lab machines.

First, you'll need to identify targets. An `nmap` scan is probably the best way to do this, and there are multiple methods of discovering hosts on the network. I used the following command:

```bash
$ nmap -Pn -sT -A --top-ports=20 a.b.c.1-254 -oN top-port-sweep
```

Where `a.b.c` is the first three octets of the lab network's IP range. This scan gives a good overview of the services running on each machine, and will help you identify low-hanging fruit.

I kept track of live hosts and the information I knew about them in a checklist in Obsidian, and ticked them off as I did them. If I liked the look of a box (either because it looked easy or had an interesting service) I highlighted it in bold.

The OffSec [Learning Path](https://help.offensive-security.com/hc/en-us/articles/360050473812) is also a great resource for identifying targets that I only found out about towards the end of my lab time. I highly recommend using it if you're not used to compromising machines and need a bit of guidance, or if you just want to make sure you've done labs that cover everything in the course material.

Secondly, you should **challenge yourself** - try to pick lab machines that cover topics you're unfamiliar with (it's better to learn them while you have access to a forum if you get stuck than it is on the exam). To find a box that covers a specific topic, you can search the topic in the forum and then do any boxes where it's mentioned.

Finally, try to do as much as you can without hints - but equally don't be scared to check the forums. You won't be able to find hints on the exam (unless you're cheating), and doing everything without hints while you're practicing will give you the best methodology. However, it's understandable if you're new to Cybersecurity (or even working on a hard machine) that you might just not be able to get any further - in this situation there's no sense handicapping yourself. Check the forum and finish the box - at the end of the day, you'll still have learned something.

I started asking myself "what would the forums tell me to do?" when I was stuck. Usually the answer is "enumerate more" - and usually that's exactly what you need. It's the most irritating advice to receive, but it's often the solution. If you're sure you've enumerated every port and googled every service version number, or think you have an attack vector but can't get it to work, that's a good point to check the forum. But try to search for threads on specific issues, rather than a broad overview of the box, to avoid spoilers and help improve your methodology.

## The Exam

> If you're going to pass the exam, you'll have time during it to document as you go. 24 hours is plenty of time to figure out these machines, so you can spend a few minutes here and there collecting evidence so you don't have to do it all at the end of the exam

The night before the exam I recapped the OSCP [Exam Guide](https://help.offensive-security.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide) and made notes on anything that looked important. I **highly recommend** doing this and reading it in full.

By the day the exam arrived, I was feeling pretty well prepared. I got up at 7:15, ready to start at 10:00, and reviewed my notes on Key Lessons I'd learned from the labs, what my strategy was, and crucial instructions from the exam guide.

By 09:25 I felt ready, and decided to just relax before I logged in to the proctoring software. I got my ID ready along with the scanned copy, opened up Google Drive on Kali ready to transfer across my connection pack, and set up my Tmux terminals. I think this *would* have calmed my nerves, had the next hour not been do disastrous.

I logged in just before 09:45, and saw no proctor. When they hadn't arrived by 09:50, I sent a very panicked email to the proctoring inbox, followed by a [Live Chat](https://chat.offensive-security.com/) request (I highly recommend using this if you have an issue in the exam).

It turned out my proctor was also logged in, and had been since 09:45, but we couldn't see each other. I eventually got connected at about 10:10, and began the pre-exam check in. I won't go into details to make sure I don't compromise the process, but I will say **make sure your webcam can scan your ID** before the exam. I did check this, but the alternative I had prepared also didn't work, so make sure you have a digital scan that you can show to them via screen-share also.

Throughout these difficulties, and even after passing the exam, OffSec support has been amazing. They were very quick to respond, and even gave me a time extension. This wasn't an ideal start, but I was determined to do well regardless.

I finished my pre-exam checks by 10:25, connected, and started the exam properly at 10:33 after reviewing the control panel and objectives. This was my strategy:
- Scan all machines with `autorecon` immediately
- Tackle the Buffer Overflow
- Tackle the rest of the machines in point-value order; in theory, this would mean I could get 75 points without having to touch the non-BOF 25-point machine
- If you can't get any of the other machines in point order, try the 25-point machine early so you have enough time to work on it
- If you can't get the 25-point machine, panic

By 11:24 I had shelled the Buffer Overflow machine, and had my first 25 points. This was a much needed confidence boost, and just goes to show that with enough practice on the Buffer Overflow content you can do it very quickly in the exam. I took some screenshots of what I'd done, submitted the flag, and went for a break.

At 11:34 I came back and finished documenting the BOF, then reviewed my AutoRecon results. I quickly found what I thought was a quick win on the 10-point machine, and battled with it until 12:30 through what appeared to be stability issues. I took a frustrated break until 12:55, tried again, and moved on at 13:18.

This was a good example of knowing when to take a step back from a box. Despite failing to get a shell, I was confident I was on the right track and could eventually get the 10-pointer, so decided to continue with my point-order strategy.

I found an exploit for the first 20-point machine by 14:04, and spent a long time trying to get an interactive shell (which is required to get the points for the machine). I took a break between 14:30 and 14:49, then finally managed to get a proper shell by 15:30. I had finished the 20-pointer by 16:06.

I noticed my progress had slowed, but I was still feeling confident. I had a difficult decision to make on 45 points; I could either go for the 10-point and remaining 20-point machine to safely pass, or try the 25-pointer and be right on the borderline with no leeway if I messed up my documentation. I wasn't sure how to debug the 10-point machine, and decided if I got the 25-pointer I could then attempt to finish the 10-point machine or get a foothold on the 20-pointer to push me into a safer number of points.

I cleaned up my terminals and documented the first 20-pointer, then started the 25-pointer at 16:20. I had found a vulnerability by 16:55 and was feeling confident! I leveraged this for remote code execution by 17:17, took a victory break between 17:26 and 17:42, then spent almost two hours trying to get an interactive shell. I was sensing a frustrating theme, and stopped at 19:19 to get some food.

I was back at 19:50, and started some manual scans on the final 20-pointer while I played with the 25-pointer. I was starting to panic, and couldn't find a vector for an interactive shell. All the usual tricks I had on my cheatsheets had failed, and I desperately started reviewing the other version numbers on the machine to look for an alternative exploit. I found an exploit that seemed unlikely to work, and noted it down as a potential alternative. I took a ten-minute break at 22:25 after working flat out for a long time, and spent a bit of time cleaning up my PC terminal and browser tabs that had accumulated.

I had another look at the 10-pointer, and tried a few alternative implementations of the exploit I was looking at. I even tried re-implementing a Metasploit module to avoid using my allowance, but had no luck.

At this point I had identified two potential exploits; one for the 10-pointer, and one for the 25-pointer. I didn't know what my other options were, and knew that if I tried an exploit and it didn't work I had very few other options. I went to bed at 00:45 full of anxiety and indecision, thinking I'd failed.

I was up at 06:25 the next morning, and reconnected by 06:50. I did some manual testing of the potential exploit on the 25-pointer to check its feasibility, knowing that if it worked I would pass, but if I instead used my time to debug the 10-pointer and it worked I'd still need one more machine. By 07:16 I decided that unfortunately the 25-pointer exploit was infeasible, and switched my focus to the 10-point machine.

When it didn't work, I felt a wave of anxiety. I had no idea what else to do for either of the boxes, and had barely enumerated the 20-pointer. I went back to my clunky, non-interactive shell on the 25-pointer. I knew it was my best bet, and had to pull it out of the bag.

I went back to my methodology checklist and, with some creative thinking, found a vector for a shell on the machine. By 08:27 I had low-privilege access, and had full access to the machine by 08:48!

After spending several hours with no idea what to do, I went from 45 points to a passing mark of 70 in under an hour. I was ecstatic, but knew I had to properly document everything I'd done to pass.

I spent the next hour collecting screenshots and reviewing my documentation for every machine against the objectives in the control panel and the Exam Guide. Once I was sure I had everything I needed, I checked out the 20-point machine to see if I could get a foothold with some low-hanging fruit and push myself to 80 points. I had a 30 minute extension, but couldn't find any way into the box. I decided to call it at 10:10, and ended the exam.

I had finished, in theory, with 70 points; this meant I had passed as long as I didn't make any mistakes on the report.

### The Report

I was exhausted, and didn't start the report until 13:00. I took another short break at 14:00 after outlining the machines, then started on the BOF writeup. This was finished by 16:00, and I took another break from 17:30 - 18:00 for tea.

I then had to deliver a session for SESH from about 18:25, so didn't restart the report until about 21:40. I had a panic moment when I was unsure about whether my exploit had met the requirements for the documentation, but knew I couldn't change anything at this point. I worked till 00:30, and got up at 07:20 to finish it off.

I'd finished the report by 09:20 after triple-checking all the requirements, and converted it to a PDF and submitted it by 09:25. My report ended up being 98 pages long, and I didn't even finish all the machines. It was this long because I included a large amount of screenshots, knowing that I was on the boundary of a pass and wanted to make sure I didn't miss *anything*.

I highly recommend practicing the process of checking the MD5 hash of your report when doing a practice exam. I also recommend avoiding automated scripts unless you've practiced using them, after reading a [horror story](https://www.reddit.com/r/oscp/comments/q03m00/failed_oscp_with_75100_points_my_advice) about them going wrong.

By 09:35 the next day (which I spent desperately trying to catch up on the lectures I'd missed doing the exam) I received my email saying I'd passed... which I completely missed and didn't read until nearly 19:00 that evening :D

### Exam Tips

You'll find plenty of exam tips by browsing the OSCP subreddit, and it's hard to keep track of them all. I think these are the most crucial things to remember:
- Take breaks, and set timers; if you're working and not getting anywhere for more than half an hour, it pays to step back from the computer and take a break. Get some fresh air, and you'll almost definitely think of a new idea. Similarly, take a break to celebrate when you make progress on a box! It's sometimes good to keep up the momentum, but if you finish a machine then that's a good time for a rest
- Regularly review the information you have for a machine by writing down the key services, known version numbers, and any known credentials. These machines (and certain enumeration scripts) can produce a lot of information in the reconnaisance stage, and it's helpful to filter through it early so you can come back for a guide when you're stuck
- Document as you go - take screenshots of key moments in the machine, such as when you run a command to gain a shell, when you receive the shell, and of course when you read a proof file. The worst situation would be to finish the exam and have your VPN access cut off, then realise you forgot something. If you document as you find things, there's a much lower chance of forgetting a crucial piece of evidence. Equally, just because a screenshot of a key command or piece of information isn't *required* as per the Exam Guide doesn't mean that it won't be useful when creating a clear report
- To avoid exam machine spoilers I didn't discuss my use of Metasploit in my writeup; however, in general I would say don't be afraid to use it, but make sure that you have evaluated all other potential options for each box before you decide your target. Pick the machine that you think you have the least chance of exploiting manually

## Things I Wish I'd Done

Here are some of the things I wish I'd done to be better prepared, or make the most of my paid lab time:
- Downloaded the various resources from the dedicated lab clients; I won't say what they are, but there are plenty of useful things on these machines that I wish I'd harvested for extra practice after my labs ended
- Completed the full [TryHackMe Buffer Overflow Prep](https://tryhackme.com/room/bufferoverflowprep) room
- Created comprehensive cheatsheets based on my lab writeups, with a markdown note per topic
- Made a list of common Windows local exploits, and which had worked for me in the past, so I could easily reference this during privilege escalation
- Made a cheatsheet of Active Directory commands to use at each stage when attacking a forest
- Completed some more network pivots into the private lab networks
- The rest of the TJNull HTB machines list
- More Linux Buffer Overflow practice, such as [Node](https://0xdf.gitlab.io/2021/06/08/htb-node.html) and [this VulnHub room](https://www.vulnhub.com/series/stack-overflows-for-beginners,198/)
- The several privilege escalation and ethical hacking courses I bought from [TCM Academy](https://academy.tcm-sec.com/), which are very well-reviewed but untouched

As you can see, there's a lot more I *could* have done to prepare, and **I still passed**.

The truth is, there are plenty of ways to prepare for this exam, and everyone will tell you something different. Reading OSCP Journey posts constantly will only overload you with information. If you take anything from this post, let it be these tips:
- Prioritise the labs over the course content, and don't underestimate how much lab time you'll need; it's worth the extra money to not need to pay for a resit
- Document for your report as you go during the exam, and learn to do this by documenting machines while you practice
- Learn how to complete a machine without taking hints, and how to dig yourself out of a rabbit hole; practice figuring out when is a good time to brute force, when is a good time to run an automated script to find stuff you may have missed, and when is a good time to step away from your computer and take a walk
- Do a practice exam! This will help you reinforce *all* of the lessons from the previous tips

The point is, don't worry too much if you can't match the level of preparation on someone else's "OSCP Journey" post. Some people complete over 60 lab machines and still fail, and some complete less than 20 and pass. Some people may have years of experience in the industry, but there are many people who pass with no prior experience. It all depends on how much you get out of the preparation you did; if you don't take any notes, or lived off hints during the labs, you're probably less likely to pass.

## What Next?

I learned a lot from this certification, and still don't know what I want to do when I leave University. I'm not even sure I want to do penetration testing, which seems like a ridiculous thing to say after completing a 24-hour penetration testing exam.

The point is, the technical skills from OSCP are highly transferable (within infosec, at least). The non-technical skills, such as persistence, report-writing, and the 'try harder' mindset, are even more so.

Whatever I end up doing, whether it's threat modelling or cybersecurity education, I'm sure this certification will be extremely helpful. And, if not, I at least had fun.

As for other certifications, I *may* have slightly caught the OffSec addiction, but I'm going to focus on my degree (and earning some money to finance other potential courses) before I try the next cert.

With all that said, if you can afford to do the OSCP I highly recommend it and I hope this post helps you succeed. Thanks for reading!

## Discussion

I also [posted this blog](https://www.reddit.com/r/oscp/comments/q9emq8/i_passed_with_70_points_my_experience/) on Reddit, and answered some questions there about the post - check it out if you have any questions!