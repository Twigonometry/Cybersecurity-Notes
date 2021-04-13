# Cybersecurity Notes
My Markdown notes for all things cybersecurity. Best viewed in Obsidian.

Read on to find out how to install this repository, and where to start finding the information you need.

**DISCLAIMER:** These notes are for educational purposes only. Use them as a learning resource or a reference guide when performing tests with *explicit permission*. I'm sure you've seen similar disclaimers on Cybersecurity resources before, but always make sure you have permission to do what you're doing. I am not responsible or liable if you misuse this resource and get into trouble.

## Installing Obsidian

### On Windows

Go to [the download page](https://obsidian.md/download)... and click Download

### On Kali

Go to [the download page](https://obsidian.md/download), and download the AppImage. Put it in any directory you want (I went with `~/Applications`)

You can either double click the file to run it, or run it with `/path/to/Obsidian-0.11.9.AppImage`

You may get the following error while running:

```bash
┌──(mac㉿kali)-\[~/Applications\]  
└─$ ./Obsidian-0.11.9.AppImage  
\[2122:0327/193255.690087:FATAL:setuid\_sandbox\_host.cc(158)\] The SUID sandbox helper binary was found, but is not configured correctly. Rather than run without sandboxing I'm aborting now. You need to make sure that /tmp/.mount\_Obsidi1nvAuD/chrome-sandbox is owned by root and has mode 4755.  
Trace/breakpoint trap
```

To fix this, run obsidian with the `--no-sandbox` flag

I setup this alias in `~/.bashrc`:

```bash
alias obsidian="~/Applications/Obsidian-0.11.9.AppImage --no-sandbox"
```

Finally, if Obsidian stops responding on launch, update Kali:

```bash
sudo apt update
sudo apt full-upgrade -y
```

## Downloading the Vault

You will need to [download git](https://git-scm.com/downloads). This is a quick and easy process.

On Windows, open the Start Menu and search for 'Git Bash', then click it to open a bash terminal (or navigate to the folder you want to install the notes into, then right-click and press 'Git Bash Here'). On Unix, open a terminal of your choice.

In this Git Bash/terminal, type the following:

```bash
git clone git@github.com:Twigonometry/Cybersecurity-Notes.git
```

If you are uncomfortable using the command line, you can install [GitHub Desktop](https://desktop.github.com/) instead.

Once you have cloned the repsitory, open Obsidian and click `Open folder as vault`, then select the `Cybersecurity-Notes` folder that was just created by Git. You're ready to go!

## Where to Start?

If you have no idea what you're looking for, go to the [[Starting Point]] page!

If you want to find writeups, they're all in the `Writeups` folder. Cheat sheets are in the `Cheat Sheets` folder. You get the idea.

The Cheat Sheets folder has a number of subfolders, such as sheets on Web Hacking, Linux, and Password Cracking. General cheat sheets, such as the [[Fundamental Skills]] cheat sheet, are in the top level of the folder. This is a good place to start if you're a complete beginner.

**What is this Repository?**

This is a public version of my main Obsidian vault, containing fleshed out cybersecurity notes and finished writeups. I will commit stuff to this repository as I finish it, moving it over from my main vault.

**Requesting Content**

If you want to request I add something to this repo, I will do my best to research it and write up some content - you can [open an issue](https://github.com/Twigonometry/Cybersecurity-Notes/issues) to make such a request. I have a [[To Add|list of things to add]], so please check it isn't there first.

## Using Obsidian

You could *technically* view this information straight out of GitHub, as it is all Markdown files - but it's built to be viewed in Obsidian, where all the code is pretty and the links between notes actually work.

You can see how this collection of notes has developed over time in the git history. Type `git log` to see a list of commits, and `git checkout [HASH]` to go back in time and see the state of the repo back then.

### Useful Hotkeys

Obsidian supports a wide range of hotkey commands. Some of the most useful ones are listed below, if you're into your Zettelkasten power use (who isn't?)

**(\*) indicates a custom hotkey**

Global Search: `Ctrl + Shift + F`

(\*) Open Random Note: `Ctrl + R`

Turn line into Checklist/Toggle Status: `Ctrl + Enter` (one press turns into list, two presses into checklist, three presses toggles status)

Toggle mode: `Ctrl + E`

(\*) Toggle default mode: `Ctrl + Shift + E` (useful for when you want to go into graph view and jump around notes, but stay in preview mode - this behaviour [is not default](https://forum.obsidian.md/t/not-retaining-preview-mode-when-switching-to-graph-view-and-back/3080/2))

(\*) Open local graph: `Ctrl + L` (this is useful for exploring linked notes - it will open the graph in a new pane, and clicking a linked note will open it along with its local graph)

Paste Plain Text: `Ctrl + Shift + V` (avoids escaping characters in nmap/autorecon output etc - useful if you want to fork/edit this repo)

(\*) Split Pane Vertically: `Ctrl + Alt + V`

(\*) Split Pane Vertically: `Ctrl + Alt + H`

Back/Forward: `Alt + Left-Arrow`/`Alt + Right-Arrow`

### Links

[Obsidian Help (Docs)](https://help.obsidian.md/Index)

[Features Overview](https://obsidian.md/features)

[Medium Guide](https://medium.com/swlh/take-better-notes-with-this-free-note-taking-app-that-wants-to-be-your-second-brain-1a97909a677b)

[What exactly is a Vault?](https://forum.obsidian.md/t/what-exactly-is-a-vault/4369/2)

[Working with Multiple Vaults](https://help.obsidian.md/How+to/Working+with+multiple+vaults)

[Referencing a Vault from another Vault](https://www.reddit.com/r/ObsidianMD/comments/hhat70/reference_a_vault_in_another_valut/)

[Single-Vault Philosophy](https://forum.obsidian.md/t/one-vault-vs-multiple-vaults/1445)

[Zettelkasten Method](https://medium.com/@rebeccawilliams9941/the-zettelkasten-method-examples-to-help-you-get-started-8f8a44fa9ae6)

[Toggling Checklists Hotkeys](https://forum.obsidian.md/t/set-hotkeys-for-creating-unordered-lists-ordered-lists-and-task-lists/4332/25)

[Key Mapping Diagram](https://keycombiner.com/collections/obsidian/winlinux/)

[Hotkeysplus Repo](https://github.com/argenos/hotkeysplus-obsidian)