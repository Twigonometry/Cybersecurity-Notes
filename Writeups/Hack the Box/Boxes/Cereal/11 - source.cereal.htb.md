# source.cereal.htb

Visiting `source.cereal.htb` in the browser gives us this page:

![[Pasted image 20210605184402.png]]

This is interesting - it gives us some potentially useful information:
- The version string: `Version Information: Microsoft .NET Framework Version:4.0.30319; ASP.NET Version:4.7.3690.0 `
- And a potential full path disclosure for the application: `c:\inetpub\source\default.aspx`

The page doesn't seem to have any interactivity beyond this.