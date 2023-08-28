# Overview

This box involved exploiting a CVE in Drupal, a CMS, on a Wordpress site. From there you could get a (very temperamental) shell, and find a MySQL password in a config file. This lets us dump hashes from the database and switch to the `bruceistherealadmin` user, who can run `snap install` as root. This leads us to the 'Dirty Sock' snap exploit, which gives us a shell as a new user with root-equivalent permissions.

The initial exploit for the box wasn't difficult to find, and the first privesc was just a case of finding the right config file. But I really enjoyed the privesc to root - reading up on how the CVE works was extremely interesting, and getting it to work was challenging enough without being frustrating.

## Ratings

I rated user a 3 difficulty, as it took me a lot of fumbling around and there were some frustrating aspects of the foothold shell interactivity that made things hard, but the steps were overall very simple.

I rated root a 3 also, as it wasn't too hard to identify and recreate the exploit. It wasn't as simple as just running dirty sock, but it was very easy to take their payload and reuse it. Creating your own snap was an extra layer of difficulty, but not necessary for the root.

**Matrix:**

![[Pasted image 20210405152937.png]]

You'd like to think this isn't very real life-applicable, as many of the CVEs are quite old; however, we know that people don't patch their stuff...