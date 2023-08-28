# Website

The website seems to be a page for a note taking application called Heed.

![[Pasted image 20210419140434.png]]

There is a download link at the bottom of the page. Only the download for the Windows Version works.

![[Pasted image 20210419141139.png]]

It says the source is from codepen. I did a brief search of public pens: [https://codepen.io/search/pens?q=heed](https://codepen.io/search/pens?q=heed) - but I got no obvious results. There was also nothing interesting in the site source.

Besides the download link, no other pages seem to be linked from the main page. [[Writeups/Hack the Box/Boxes/Atom/5 - Enumeration#gobuster|Gobuster]] also threw nothing up. The only interesting Gobuster result was the `/examples` directory, which had a 503 status code.

![[Pasted image 20210419143912.png]]

I added a entry for `atom.htb` to my hosts and visited that domain to see if there was a different page. Unfortunately the page stayed the same.

![[Pasted image 20210420170704.png]]

I also checked out the `/releases` directory to see if there were any other versions of the code available:

![[Pasted image 20210421211435.png]]

There was a directory listing, but only the one file.