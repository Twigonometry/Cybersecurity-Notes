 # Key Lessons
 
 - `gobuster` does not automatically check for directories! This isn't an issue on most webservers, as often `/cgi-bin` will redirect to `/cgi-bin/`, but if not you must use the `-f` flag or another tool such as `dirsearch`
- Shellshockable file extensions include `.cgi`, `.sh`, and `.txt`