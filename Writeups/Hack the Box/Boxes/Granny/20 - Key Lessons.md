# Key Lessons

Here are some of the important things I learned:
- Making sure to check the sub-technologies running on ports like port 80 - I overlooked WebDAV for a long time!
- WebDAV as a method of file upload
- Bypassing upload restrictions via file renaming
- Google the windows server version followed by "privilege escalation" rather than just relying on searchsploit and exploit suggester!
- Having a read of the `churrasco.exe` exploit [writeup](https://github.com/Re4son/Churrasco/blob/master/DEFCON-18-Cerrudo-Token-Kidnapping-Revenge.pdf) after the box was extremely helpful, and shed some light on how `SeImpersonatePrivilege` attacks and similar exploits actually work. I now understand much better which accounts and services are likely to be vulnerable to these privesc techniques, which will be useful going forward (even if Juicy Potato didn't work in this instance) 