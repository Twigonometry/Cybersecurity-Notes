# Starting Point

If you don't know what you're looking for in this repository, or want to find an index of information, start here!

## What does this Repository contain?

This repository contains a number of resources about cybersecurity. I plan to include a number of different categories of information, grouped into folders.

- Cheat Sheets: condensed information, payloads and commands regarding specific topics. Good starting points are listed below:
	- [[Fundamental Skills]]
	- [[Basic Linux Commands]]
- Cyber Topics: detailed overviews of a specific topic, for example a specific exploit I took a deep dive into on a box, or a beginner's look at XSS
- Writeups: full rundowns of how I completed challenges on HacktheBox (HTB), TryHackMe (THM), and various CTFs
- CVEs: overviews of common CVEs I've found useful, and Proof of Concept (PoC) code where relevant
- Articles: some of my favourite cybersecurity articles - either news on big hacks, or well thought out resources that I'd feel bad trying to rip off here
- [[Useful Resources|Resource Lists]]: Similarly to the above, I can't replicate everything here - I'll point you to some of my favourite online resources for learning and research

The repository is designed to be highly connected and full of atomic notes, just the way Obsidian intended. This means I will overlap these topics, and try to link 'Cyber Topics' articles to practical examples on writeups, or link writeups to the relevant cheat sheet so you can read more about the commands used.

**Why Build This?**

I had a few main motivations:
- It was unwieldy to have a set of cheatsheets online that you had to find the links to and reopen every time you wanted to look up a command
- I was sick of having to open multiple Cherry Tree files to reference my writeups - for example, when completing a new box that was similar to an old one
- I wanted to be easily able to link cheatsheets to practical examples from writeups

Obsidian easily solves all of the above issues (well - with some effort from me)

## Who is this Repository for?

I want this repository to be a useful resource for people of all skill levels in Cybersecurity. With [Sheffield Ethical Student Hackers](https://shefesh.com) and through my work I've had a chance to develop my skills communicating the basics of Cybsersecurity, and I hope to be able to provide a good resource here for complete beginners.

I also want to do some deep dives into some of my favourite topics. Unfortunately I am still somewhat of a beginner myself, and still grappling with some of the harder concepts. Hopefully, as my cyber knowledge evolves so will this resource, and it will become more relevant for those of you who can root a Hard Windows box before it leaves the Release Arena.

For those of you interested in reading writeups, I'll be putting all of mine on here. I will only upload writeups of retired boxes on HacktheBox (as per policy), so they will trickle through as I move them from my private vault. I'll also look at uploading TryHackMe solutions and solutions for any CTFs I take part in.

I also want to include resources for developers who are looking to improve the way they write secure code. Stay tuned on this one - I will be looking for examples of badly implemented sites in CTF and HTB challenges, and uploading remedied code here when I can. I'd also like to include resources for secure use of Cloud Computing platforms such as AWS.

In summary, these are the areas that are relevant:
- For Beginners:
	- [[Fundamental Skills]]
	- [[Basic Linux Commands]]
- For Learning Reconnaisance:
	- [[Reconnaisance]]
	- [[Common Ports]]
- For writeups and CVEs:
	- `/Writeups` folder for all writeups
	- [[Vulnerabilities Index]] for all CVEs
- For learning resources and tools:
	- [[Useful Resources]]

So, that's the plan for the repo. 

## How do I find stuff?

Information is organised into folders to make things a little easier to find. Some people build Obsidian vaults completely flat and rely entirely on links, but I like having folders to point people to specific areas. Unfortunately, folders cannot be linked to. Folders are also not represented on the graph, and notes in folders will not tend to group towards each other (although I wish they did).

There is no central list of links to specific notes, but many notes contain multiple links to other notes. I've experimented with building 'index' nodes, and may do so in the future, but it is easy enough to find notes for now. The main reason for not creating an index note is that it would be hard to maintain, but some specific categories have indexes (and all writeups have an index note).

The global search (`Ctrl + Shift + F`) is the easiest way to find a specific topic. For example, searching `python webserver` will display all the instances in our notes where we started a python server. You can also search notes by their tag in the global searchbar. For example, searching `tag:#malware` will display all notes with the malware tag.

### The Graph

If you want to get a feel for how the information links together, you can view the Graph. Open it by pressing `Ctrl + G`, or by clicking the graph icon on the far-left of the screen. The graph is a good indicator of what topics are linked together, and highly connected nodes are probably the nearest thing to an 'index' page that may serve as a good launching point into other knowledge areas.

Within the graph settings, you can enable showing tags, turn off orphan nodes (i.e. only show connected notes), and search for files. Groups can also be created to display subsets of notes (I haven't got this far yet).

Enabling tags, for example, links all notes by category rather than by their backlinks:

![[Pasted image 20210501111234.png]]

Clicking a tag on the graph will open a list in the searchbar of all notes with that tag.

You can also open a local graph by pressing `Ctrl + L`. This will show the notes that are connected to the note you are currently viewing. You can use this to browse connected notes easily - clicking a new one will open its local graph, so you can traverse chains of related notes. Use `Alt + Left-Arrow` and `Alt + Right-Arrow` to navigate back and forth.

### Exploring

If you want to dive in blindly, you can! Press `Ctrl + R` to view a random note. This is good for rediscovering a topic you may not have looked at for a while, refreshing your memory, and potentially finding notes in the depths of the repo that I should have fleshed out but haven't. You can see a list of useful hotkeys in the [[README#Useful Hotkeys]] section.