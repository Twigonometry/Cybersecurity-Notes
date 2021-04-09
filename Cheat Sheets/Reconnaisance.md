# Reconnaisance
## Autorecon
**Basic Syntax**

```bash
autorecon [IP]
```

### Running a Webserver

Once your scans are finished, you can easily view the results by starting a webserver in the `/results` directory and visiting it on localhost:

```bash
┌──(mac㉿kali)-[~/Documents/HTB/BOX/results/]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Then visit `http://localhost:8000` and view the results in browser.