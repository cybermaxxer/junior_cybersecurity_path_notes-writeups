# how to remember shit when bashing or powershelling

**tl;dr:** don't memorize syntax, memorize your way of finding it. remember the intent ("list services", "find big files"), guess the command from naming patterns, look it up in under 10 seconds, then save what you looked up twice. keep about 20 to 30 daily commands in your head and let tools hold the rest.

## 1. the core idea (why this works)

| fact | why it matters |
|---|---|
| knowing what you need is the skill | the flag is just a label somebody picked |
| most flags are human convention | there's no deep logic to derive them from, so memorizing feels like pulling teeth |
| your working memory is small | spending it on arbitrary syntax is a bad trade |
| pros look stuff up all day | looking up is part of the job, not a weakness |

the goal is to move syntax into external memory (help, history, cheat sheet) and keep your brain for reasoning.

## 2. the 3 step system: GLS

mnemonic: **GLS** = guess, lookup, save.

| step | what you do | example |
|---|---|---|
| **G**uess | use naming patterns to predict the command | need to stop a service, so try `Stop-Service` |
| **L**ookup | if the guess fails, find it fast with built in tools | `Get-Command *service*` or `tldr tar` |
| **S**ave | if you looked it up twice, put it in your cheat sheet | one line, indexed by goal |

## 3. bash toolkit

mnemonic: **TMH** = tldr, man, help. zoom in only as far as you need.

| need | command | why it works |
|---|---|---|
| quick examples | `tldr tar` | "too long; didn't read": a short digest of the 5 things people actually do |
| quick flag reminder | `tar --help` | most gnu tools support it. `--word` is the readable long form, `-w` is the short alias |
| full manual | `man tar` | `man` = manual |
| search inside a man page | `/EXAMPLES` then `enter`, `n` next, `q` quit | man opens in a pager called `less`, where `/` means search. same key as vim, just convention |
| don't know the command name | `apropos "copy files"` | searches the one line descriptions of all man pages. the name is french "à propos", meaning "regarding" |
| one line summary | `whatis tar` | prints just the short description |
| web fallback | `curl cheat.sh/tar` | curl fetches a url, cheat.sh answers with examples as plain text |

install tldr once: `sudo apt install tldr`, then `tldr -u` to fetch the pages.

### history is free memory

you already typed most of what you need once.

| keys or command | what it does | why |
|---|---|---|
| `ctrl+r` | search your history as you type | `r` = reverse search. press it again to jump to the next older match |
| `history \| grep ssh` | list past commands containing "ssh" | pipe sends history text into grep, which filters lines |
| `!!` | repeat the last command | "bang bang", convention |
| `sudo !!` | rerun the last command as root | classic fix for "permission denied" |
| `!$` | last argument of the previous command | `$` means "end", so it's the end of the last line. convention |

## 4. powershell toolkit

mnemonic: **CMH** = command, member, help.

| cmdlet | answers | example |
|---|---|---|
| `Get-Command` | what cmdlet does this job? | `Get-Command -Noun Service` |
| `Get-Member` | what properties does this output have? | `Get-Service \| Get-Member` |
| `Get-Help` | how do i use this cmdlet? | `Get-Help Get-Service -Examples` |

### the naming pattern is real, not a mnemonic

every cmdlet is `Verb-Noun`, and the verbs come from an approved list (run `Get-Verb`). microsoft enforced this on purpose so you can predict names. learn one noun and you get the whole family:

| noun | get | start | stop | restart |
|---|---|---|---|---|
| Service | `Get-Service` | `Start-Service` | `Stop-Service` | `Restart-Service` |
| Process | `Get-Process` | `Start-Process` | `Stop-Process` | n/a |

### the help zoom levels

mnemonic: **EPF** = examples, parameter, full.

| switch | shows | when |
|---|---|---|
| `-Examples` | real usage samples | always first, fastest payoff |
| `-Parameter Name` | docs for one parameter, wildcards allowed (`-Parameter *`) | you know the command but forgot a flag |
| `-Detailed` | description, parameters, examples | middle ground |
| `-Full` | everything, including input and output types | deep details |
| `-Online` | opens the web docs | freshest version, best formatting |

run `Update-Help` once in an admin shell, otherwise help is often a skeleton. the reason: help files are downloaded on demand so microsoft can fix docs without a windows update.

### other powershell lookup tricks

| need | command | why it works |
|---|---|---|
| concept help (pipes, operators, loops) | `Get-Help about_Comparison_Operators` | `about_` topics are essays on concepts |
| list all concept topics | `Get-Help about_*` | wildcard |
| see valid parameters while typing | `ctrl+space` after a `-` | the shell lists them |
| prefer a gui form | `Show-Command Get-Service` | builds a clickable form |
| tab completion | press `tab` | finishes cmdlet names and parameters, so you rarely type them in full |
| past commands | `Get-History`, or `ctrl+r` in PowerShell 7 | same idea as bash history |

## 5. reading a syntax block

the same symbols show up in man pages and powershell help.

```
Get-Service [[-Name] <string[]>] [-ComputerName <string[]>]
```

| symbol | means | why |
|---|---|---|
| `[ ]` | optional | shared convention across man pages and powershell |
| `[[-Name] ...]` | whole thing optional AND the name `-Name` can be skipped | inner bracket makes the name optional, outer makes the whole parameter optional |
| `<string[]>` | the type it expects. `[]` means it can take several values | like a list |
| no brackets | required | you must provide it |
| `...` (bash) | can repeat | `[FILE]...` accepts many files |
| `UPPERCASE` (bash) | placeholder you replace | `OPTION` isn't typed literally |

so `Get-Service spooler` works, because `-Name` is positional.

## 6. bash vs powershell, side by side

| goal | bash | powershell |
|---|---|---|
| get help with examples | `tldr cmd` | `Get-Help cmd -Examples` |
| find a command by topic | `apropos "topic"` | `Get-Command *topic*` |
| list files | `ls` | `Get-ChildItem` (alias `ls`, `dir`) |
| filter output | `... \| grep text` | `... \| Where-Object Prop -eq 'x'` |
| list processes | `ps aux` | `Get-Process` |
| list services | `systemctl list-units --type=service` | `Get-Service` |
| search history | `ctrl+r` | `ctrl+r` or `Get-History` |
| see what's inside the output | read the text | `\| Get-Member` |

the big difference: bash pipes plain text, so you parse it with grep and awk. powershell pipes objects with properties, so you filter by property name instead. that's why `-eq` exists: `>` and `<` are already taken by redirection, so comparisons became word flags (`-eq`, `-ne`, `-gt`, `-lt`).

## 7. build your own cheat sheet

rules that make it actually work:

| rule | why |
|---|---|
| index by goal, not by tool | you remember "find open ports" before you remember "nmap" |
| one line per entry | a wall of text won't get read |
| add only things you looked up twice | keeps it short and relevant |
| include a one word reason for each flag | builds understanding, not just a paste |
| keep it in one searchable file | `grep -i "ports" cheatsheet.md` finds it instantly |

example entries:

| goal | command | note |
|---|---|---|
| open ports plus service versions | `nmap -sV -p- 10.10.10.5` | `-sV` = scan, version. `-p-` = all ports |
| extract a tar.gz | `tar xf archive.tar.gz` | `x` = extract, `f` = file |
| find files changed in last day | `find . -mtime -1` | `-mtime -1` = modified less than 1 day ago |
| running services only | `Get-Service \| Where-Object Status -eq 'Running'` | pipe passes objects, `-eq` = equals |

### shortcuts for your own favorites

```bash
# bash: put this in ~/.bashrc
alias ports='nmap -sV -p-'   # alias name is anything you choose
```

```powershell
# powershell: edit your profile with: notepad $PROFILE
Set-Alias gs Get-Service
```

aliases are arbitrary, so only create them for things you type daily.

## 8. the memory rules (adhd friendly)

| rule | detail |
|---|---|
| core set of 20 to 30 | `ls`, `cd`, `grep`, `find`, `curl`, `ssh`, `chmod`, `Get-Command`, `Get-Help`, `Get-Member`. repetition makes these stick without flashcards |
| everything else is a lookup | low payoff per memorization hour |
| learn by doing | overthewire, tryhackme, hackthebox. labs repeat the core set for you |
| copy, then break it down | never paste blindly, ask what each part does once |
| notes are legit | many practical exams (oscp for example) allow your own notes |

## 9. full workflow in one glance

1. name the need in plain words ("show disk space")
2. guess the command from the pattern (`Get-` for powershell, or `apropos` for bash)
3. get examples first (`tldr` or `-Examples`)
4. check one flag only if needed (`man` then `/flag`, or `-Parameter`)
5. run it, and save the line if it's the second time you looked it up

## feynman check

1. you forgot the cmdlet to restart a service. what do you try, and why does it work?
   * guess `Restart-Service`, or run `Get-Command -Noun Service`. it works because every cmdlet is verb then noun, so the noun gives you the whole family.
2. you remember a linux tool exists for "show disk space" but forgot its name. what do you run?
   * `apropos "disk space"`. it searches descriptions, so you find the command (`df`) from the job it does.
3. when does a command deserve a spot in your cheat sheet?
   * when you've looked it up twice. once is noise, twice means it's a real part of your workflow.
