# Overview
In preparation for my OSCP, I have begun the long journey of completing all retired boxes, sorted by user-rated difficulty. Starting on the 1st May 2021, I am doing *Blue*.

*Note:* I am doing these retired boxes for now on our shared [SESH Account](https://www.hackthebox.eu/profile/451740) - because why pay for VIP twice?

I have been recommended to tackle these up to about a 6 in difficulty, as that is about the highest I should encounter in the exam. I don't expect to do them all by the time I sit my exam (aiming for mid-late August 2021), but I have to start somewhere. I expect to learn a lot, especially about Windows which is currently my weakest area.

Blue has a difficulty rating of 2.4, the lowest on HTB:

![[Pasted image 20210501114708.png]]

---

The box was super simple - all it involved was enumerating the box to discover SMB was running and that the OS was Windows 7, vulnerable to Eternal Blue.

If you want to root it quickly, you can just use metasploit. I tried to manually exploit it, as Metasploit is not allowed on OSCP. It took an extra couple of hours to setup and get the right details, but was still pretty easy - just fiddly.

All in all it only took about 3 hours. Pretty slow going for a box of this difficulty, but I will hopefully speed that up - and now I'm equipped with a working Eternal Blue exploit I can reuse that on other boxes :)

## Ratings

I rated the box a 1 for both user and root - the exploit was simple really, made harder only by the fact I tried to do it manually. And there was no priv esc involved, as it dropped us straight in as `SYSTEM`.

# Tags

#writeup #windows #cve #smb #oscp-prep 