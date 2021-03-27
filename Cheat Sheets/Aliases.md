# Aliases

## Useful Aliases

```bash
# Running VPNs
alias htbvpn="sudo openvpn ~/Documents/HTB\ Access/Twigonometry.ovpn"
alias ravpn="sudo openvpn ~/Documents/HTB\ Access/release_arena_Twigonometry.ovpn "
alias thmvpn="sudo openvpn ~/Documents/THM\ Access/Twigonometry.ovpn"
alias seshvpn="sudo openvpn ~/Documents/HTB\ Access/shefesh.ovpn"

# Edit common files
alias nanbash="nano ~/.bashrc"
alias nanhosts="nano /etc/hosts"

# Start webserver in enum directory
alias enumserve="cd ~/Documents/enum; python3 -m http.server"

# Run ghidra, deleting cache file to eliminate startup issues
alias run-ghidra="rm -r ~/.ghidra; /opt/ghidra/ghidra_9.2.2_PUBLIC/ghidraRun"

# Go to common directories
alias sesh-site="cd ~/Documents/SESH/SeshWebsite"
alias sesh-serve="cd ~/Documents/SESH/SeshWebsite; bundle exec jekyll serve"
alias raspictf="cd ~/Documents/SESH/RaspiCTF"

# Run obsidian
alias obsidian="~/Applications/Obsidian-0.11.9.AppImage --no-sandbox"
```

## Adding Aliases

```bash
nano ~/.bashrc
[...type your code...]
source ~/.bashrc
```