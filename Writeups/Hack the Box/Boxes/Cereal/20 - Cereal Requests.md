# Cereal Requests

Now we've picked the code apart a little, we can try interacting with the site.

If we go to `/requests` in our browser and pass it to Burp, we can capture the request and then change the request method:

![[Pasted image 20210406085414.png]]

Press `Ctrl + R` to send to repeater, then right-click and select 'Change request method' to turn it into a POST request. We also need to set the `Content-Type` header to `application/json`, as the app expects JSON and otherwise responds with `415 Unsupported Media Type`.

![[Pasted image 20210406085814.png]]

Let's add our token to the request. We add the following header: `Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTgzMDA5ODMsIm5hbWUiOjF9.VgWvwKp0RMrr4NLnJxnIWoUJII3JQlUJecyFVpDlXvo` and resubmit the request.

*Note:* This is just the token I used when doing this box - it will expire after seven days, so you will need to generate your own using `gentoken.py` if you are doing this box.

![[Pasted image 20210406090351.png]]

We are no longer unauthorised :)

Now we need to craft a valid request. Experimenting with some JSON input gives us some clues about how to structure the request - namely, that the `"JSON"` field is required:

![[Pasted image 20210406090615.png]]

Submitting some more JSON reveals more about how it should be structured:

![[Pasted image 20210406091211.png]]

It seems to need `"` characters to be escaped - there were some clues about this in the source code, for example `var header = "{\\"typ\\":\\"JWT\\",\\"alg\\":\\"HS256\\"}";`, but just to be sure I formatted my payload by running it through the `JSON.stringify()` method used by the website.

I created the JS file `test-www/stringify.js`:

```javascript
console.log(JSON.stringify({ JSON: JSON.stringify({title:'t',flavor:'f',color:'#FFF',description:'d' }) }))
```

And ran it from a very simple HTML file:

```html
<html>
    <head>
        <script defer src="./stringify.js"></script>
    </head>
</html>
```

Opened it in firefox:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/cereal/test-www]
└─$ firefox index.html
```

![[Pasted image 20210406092035.png]]

This gives us the correctly-formatted payload:

```
{"JSON":"{\"title\":\"t\",\"flavor\":\"f\",\"color\":\"#FFF\",\"description\":\"d\"}"}
```

![[Pasted image 20210406092416.png]]

We have successfully created a cereal!

I tried to save the Burp request as a `curl` command, but it was quite temperamental. To replicate a request in `curl`, right click in repeater and press 'Copy as curl command'. I find it easier to do it in Burp Suite the first time, as it creates many of the headers for you, but for replicating it later `curl` is faster:

```bash
curl -i -s -k -X $'POST' \
    -H $'Host: 10.10.10.217' -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Connection: close' -H $'Upgrade-Insecure-Requests: 1' -H $'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTgzMDA5ODMsIm5hbWUiOjF9.VgWvwKp0RMrr4NLnJxnIWoUJII3JQlUJecyFVpDlXvo' -H $'Content-Type: application/json' -H $'Content-Length: 86' \
    --data-binary $'{\"JSON\":\"{\\\"title\\\":\\\"t\\\",\\\"flavor\\\":\\\"f\\\",\\\"color\\\":\\\"#FFF\\\",\\\"description\\\":\\\"d\\\"}\"}' \
    $'https://10.10.10.217/requests'
```