# Useful Resources
A compilation of some of my favourite Cybsersecurity Resources - tools, articles, practice platforms, learning materials, and more.

This list is based on a version that I helped to build for [Sheffield Ethical Student Hackers](https://shefesh.com) society. You can find the original list here: [https://shefesh.com/wiki/resources](https://shefesh.com/wiki/resources)

I converted the HTML code to markdown using [Turndown](https://domchristie.github.io/turndown/). Thanks for making my life easy!

### Penetration Testing Labs

[HackTheBox.eu](https://www.hackthebox.eu) - Generally more advanced boxes, however there are some easy boxes too

[TryHackMe](https://tryhackme.com/) - A useful website for walkthroughs and instructional learning (which can be hard to come by in cybersecurity). Some of our favourite rooms are linked below (and you can find a full list [here](https://tryhackme.com/hacktivities))

*   [Linux](https://tryhackme.com/room/zthlinux) - Missed our session on Linux Basics? TryHackMe has you covered
*   [Juice Shop!](https://tryhackme.com/room/owaspjuiceshop) - You'll have seen this one if you've been to pretty much any of our sessions :) It's a great start for learning web application testing
*   [Vulnversity](https://tryhackme.com/room/vulnversity) - A great room for the basics of recon, Burp Suite, and more
*   [Nmap](https://tryhackme.com/room/rpnmap) - Learn the ropes of this crucial networking tool
*   [Attacktive Directory](https://tryhackme.com/room/attacktivedirectory) - Learn some Active Directory enumeration tools and methodologies, step by step

[Immersive Labs](https://immersivelabs.online) - A collection of highly interactive labs, ranging from theory to guided tutorials of common tools - free for students!

[Vulnhub](https://www.vulnhub.com/) + Docker Machines - Vulnhub is a website full of Virtual Machine images, ready to be hacked! Specific boxes that we enjoyed are listed below, along with some cool Docker images we've used for exercises!

*   [Stack Overflows for Beginners](https://www.vulnhub.com/entry/stack-overflows-for-beginners-101,290/)
*   Brooks' Permissions Puzzle - run in a [Docker Environment](https://labs.play-with-docker.com/#) with "docker run --rm -it thelostlambda/perm-puzzle"

[OverTheWire Wargames](https://overthewire.org/wargames/) - A collection of miniature challenges, mostly Linux based, and great for learning the basics

[Hack this Site](https://www.hackthissite.org/) - A site similar to Wargames with a series of missions, in a range of difficulties

### Learning Resources

[Hacksplaining](https://www.hacksplaining.com/) - The basics of hacking with interactive examples and short quizzes

[GTFOBins](https://gtfobins.github.io/) - A website that shows possible privilege escalation vectors through SUID/GUID binaries

[ippsec.rocks](https://ippsec.rocks) - A searchable directory of hundreds of HackTheBox video walkthroughs

[OWASP Top 10](https://owasp.org/www-project-top-ten/) - OWASP's list of the most critical Cybsersecurity risks

[excess-xss](https://excess-xss.com/) - A comprehensive guide to Cross Site Scripting attacks

[Portswigger SQL Injection](https://portswigger.net/web-security/sql-injection) - An excellent writeup of SQL Injection techniques, from the people who brought you Burp Suite! Includes a great [cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

[AWS UK-OFFICIAL Quickstart](https://aws.amazon.com/quickstart/architecture/compliance-uk-official/) - AWS' Template for an Official-rated cloud network. A good example of secure cloud infrastructure!

[Scraping Club](https://scrapingclub.com/) - A great website full of web scraping challenges

[Enumerating Active Directory](https://medium.com/@Shorty420/enumerating-ad-98e0821c4c78) - An interesting article on common commands when poking around a Windows Domain Controller

#### Scripting

*   [Simple Bash Scripting](https://www.linux.com/training-tutorials/writing-simple-bash-script/) - A nice guide to writing basic bash scripts
    
*   [Bash with arguments](https://www.baeldung.com/linux/use-command-line-arguments-in-bash-script) - A guide to passing data to your script
    
*   [Python with arguments](https://www.tutorialspoint.com/python/python_command_line_arguments.htm) - Command line arguments in a python script
    

### Courses + Videos

[Udemy](https://www.udemy.com/topic/cyber-security/) - A large catalogue of Cybersecurity courses

[LinkedIn Learning - Cybersecurity Foundations](https://www.linkedin.com/learning/cybersecurity-foundations-2) - A course by Malcolm Shore, with more courses on his page

[LinkedIn Learning - Python for Automation](https://www.linkedin.com/learning/using-python-for-automation) - A course by Sam Pettus, covering the basics of web scraping, Python HTTP requests, and more

[Udemy Automation](https://www.udemy.com/course/automate/) - Another automation course

[Computerphile Password Cracking](https://www.youtube.com/watch?v=7U-RbOKanYs) - A brilliant explanation of password cracking (featuring plenty of sexy GPUs)

[Computerphile SQL Injection](https://www.youtube.com/watch?v=ciNHn38EyRc) - A great visual explanation of SQL Injection

[Computerphile Diffie-Hellman](https://www.youtube.com/watch?v=NmM9HA2MQGI) - A gorgeous visual explanation of a popular key exchange algorithm for all you cryptography nerds

[CompTIA Exam Prep](https://www.youtube.com/watch?v=qiQR5rTSshw&t=82s) - A (long) video that goes over the crucial information for the CompTIA+ Qualification - even if you're not studying for it, this video is a great intro to networking!

### Walkthroughs and Writeups

[Ippsec](https://www.youtube.com/channel/UCa6eh7gCkpPo5XXUDfygQQA) - A YouTube channel dedicated to walkthroughs of HackTheBox and other challenges

[Juice Shop Solutions](https://bkimminich.gitbooks.io/pwning-owasp-juice-shop/content/appendix/solutions.html) - A comprehensive list of solutions for the Juice Shop Challenges

[Jack Barradell-Johns](https://blog.barradell-johns.com/index.php/category/writeups/) - Excellent writeups from our former Vice President!

[WireGuard Setup](https://shefesh.com/downloads/wg-setup.pdf) - Set up your own VPN network with WireGuard!

### Tools

Remember the [Code of Conduct](https://shefesh.com/downloads/SESH%25Code%25of%25Conduct.pdf) (and the Computer Misuse Act) when using these tools! They are for education only, to be used on systems where you have explicit permission

Once you're sure you're working ethically and within the law... have fun!

#### Enumeration + Privelege Escalation

*   [Nmap](https://nmap.org/) - A tool for enumerating networks, with lots of built in scripts for enriching information - this is the first step in most security assessments!
    
*   [Gobuster](https://github.com/OJ/gobuster) - Insanely fast tool for discovering webpages on a domain - often the first step when exploring a web app
    
*   [ldapsearch](https://linux.die.net/man/1/ldapsearch) & [ldapenum](https://sourceforge.net/projects/ldapenum/) - Tools for enumerating system and domain controllers over LDAP - useful for Windows boxes!
    
*   [pspy](https://github.com/DominicBreuker/pspy) - For monitoring processes on a Linux machine - useful for discovering interesting things post-exploitation!
    
*   [PrivEsc Scripts Suite](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) - A list of brilliant scripts for enumerating ahead of privelege escalation, including linPEAS and winPEAS. (It pairs nicely with [this](https://book.hacktricks.xyz/))
    
*   [Bloodhound](https://github.com/BloodHoundAD/BloodHound) - Brilliant tool for visualising exploitation paths in Active Directory, and suggesting exploits
    

#### Networking + Web Scraping

*   [Burp Suite](https://portswigger.net/burp) - A powerful tool for capturing and analysing HTTP requests, and modifying them on the fly - this is an essential in your toolkit!
    
*   [Wireshark](https://www.wireshark.org/) - An incredibly powerful tool for analysing network traffic
    
*   [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/) - The essential web scraping library, with [great documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)
    
*   [Scrapy](https://scrapy.org/) - A powerful web scraping framework
    

#### Exploitation

*   [sqlmap](http://sqlmap.org/) - A tool for automatically detecting and performing SQL injection attacks
    
*   [Metasploit](https://www.metasploit.com/) - An extensive set of exploit implementations, downloadable for free via Metasploit Framework
    
*   [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - A mindblowingly versatile tool used for enumerating and exploiting Windows Machines and Active Directory - with incredible [documentation!](https://mpgn.gitbook.io/crackmapexec/)
    
*   [Impacket](https://github.com/SecureAuthCorp/impacket) - A collection of brilliant Python Scripts, perfect for pulling secrets out of Windows Machines (and much more). We used many of these scripts during our Enumeration Session
    
*   [tomcatWarDeployer](https://github.com/mgeeky/tomcatWarDeployer) - For deploying malicious payloads to compromised Tomcat webservers
    

#### Social Engineering + OSINT

*   [Social Engineering Toolkit](https://github.com/trustedsec/social-engineer-toolkit) - A suite of social engineering and OSINT tools, including phishing and fake login pages!
    

#### Reverse Engineering

*   [Ghidra](https://ghidra-sre.org/) - A suite of software reverse engineering tools, developed by the NSA
    

#### Utility

*   [Cyberchef](https://gchq.github.io/CyberChef/) - A GCHQ released tool that's useful for encodings, cryptography and a ton of other useful tools!
    
*   [jwt](https://jwt.io/) - A tool useful for decoding JWT tokens used in web applications
    
*   [John the Ripper](https://www.openwall.com/john/) - A great password cracking tool, supporting hundreds of hash and cipher types
    
*   [Regex101](https://regex101.com/) - A lovely little regex checker, for help with all those greps
    
*   [JSLinux](https://bellard.org/jslinux/vm.html?cpu=riscv64&url=fedora29-riscv-2.cfg&mem=256​ca) - Try Linux out in your browser! (Although we recommend [installing it properly](https://shefesh.com/wiki/virtual-machine.html))
    
*   [tmux](https://www.youtube.com/watch?v=Lqehvpe_djs) - A video guide to tmux from Ippsec, a useful tool for terminal productivity
    
*   [HTTPBin](https://httpbin.org/) - A website for testing HTTP requests
    
*   [CTF Tools](https://github.com/Twigonometry/CTF-Tools) - A work-in-progress repo with various cybersecurity tools, including a password cracker and a repeater, built by Mac
    

#### Lists within lists!

*   [Red Teaming Toolkit](https://github.com/infosecn1nja/Red-Teaming-Toolkit) - A collection of amazing repositories and tools for all your hacking needs
    
*   [SecLists](https://github.com/danielmiessler/SecLists) - Thought this list was long? This repo compiles an egregious number of passwords, URLs, and payloads for fuzzing, password cracking, and everything in between
    
*   [ExtendsClass](https://extendsclass.com/) - A host of free online developer tools for testing Regexes, API calls, XML validation, and more!
    

### Academic Research

[Security of Advanced Systems](https://www.sheffield.ac.uk/dcs/research/groups/security) - A research group @ UoS, focusing on security by design and security analysis methods

[Verification](https://www.sheffield.ac.uk/dcs/research/groups/verification) - A research group @ UoS, focusing on formal methods and mathematically rigorous verification of software and hardware

### Textbooks

[Linux Pocket Guide by Daniel J Barrett](https://www.amazon.co.uk/Linux-Pocket-Guide-Daniel-Barrett/dp/1491927577/) - A detailed list of the most useful Linux commands, and how to use them!

[Web Application Hackers Handbook by Stuttard and Pinto](https://www.amazon.co.uk/Web-Application-Hackers-Handbook-Exploiting/dp/1118026470) - An incredibly detailed book with demonstrations of a wide range of exploits

[Network Security Assessment by Chris McNab](https://www.amazon.co.uk/Network-Security-Assessment-Know-Your/dp/149191095X) - Another highly detailed book focusing on network security

[Red Team Field Manual by Ben Clark](https://www.amazon.co.uk/Rtfm-Red-Team-Field-Manual/dp/1494295504) - A Red Teamer's reference guide

### Blogs & News

[NCC Group](https://www.nccgroup.com/uk/about-us/newsroom-and-events/blogs/?Year=2020) - A series of great blogs, including the excellent 'Black Team War Stories'

[Hacker News](https://news.ycombinator.com/) - A curated list of technology news articles

[Risky Business Podcast](https://risky.biz/) - A regular podcast taking a deep dive into Cybsersecurity news

[NCSC](https://www.ncsc.gov.uk/section/keep-up-to-date/all-blogs) - The official blog of the National Cyber Security Centre

[AWS Security](https://aws.amazon.com/blogs/security/) - General security articles from AWS

[AWS Provable Security](https://aws.amazon.com/security/provable-security/) - Another AWS blog, focusing more on formal methods