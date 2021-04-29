# Overview

I did this box back in December 2020. It was the fifth box I'd done, and only the second medium-rated box I'd tried. It took me a few days of pretty non-stop work to get User, and I had Root after a week.

I wasn't as good at taking screenshots for my notes back then, so when I converted this writeup to Obsidian I made sure to go back and get some. Therefore you may see some screenshots dated after the box retired. My IP might also change between bash commands :)

---

This box was extremely fun. The initial exploit involved enumerating a webserver to discover it was linked to some AWS resources. There were then two parallel parts: interacting with a DynamoDB shell to exfiltrate some credentials, and uploading a web shell to an S3 bucket for code execution on the box.

Once you were on the box, you could use the stolen credentials to log in as the user `roy`. roy had access to a locally-hosted web app which you could access via SSH tunneling and exploit by adding a malicious entry in a database that caused the web app to read a sensitive file and convert it to a PDF.

## Ratings

I rated user a 6 for difficulty at the time, as I found the debugging of the DDB code very difficult. After revisiting the box I would probably rate it a 5, as the steps were fairly simple but just required some knowledge of AWS.

I rated root a 7 for difficulty. It involved some techniques I hadn't used before, such as SSH tunneling, and a cool custom exploitation on a web app, plus an interesting way of stealing a sensitive file via a PDF attachment which I hadn't seen before.