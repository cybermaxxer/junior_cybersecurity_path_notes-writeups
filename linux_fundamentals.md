## Table of Contents

- [1. Module Digest](#1-module-digest)
  - [1.1 Linux Structure](#11-linux-structure)
  - [1.2 Linux Distributions](#12-linux-distributions)
  - [1.3 Introduction to Shell](#13-introduction-to-shell)
  - [1.4 Prompt Description](#14-prompt-description)
  - [1.5 Getting Help](#15-getting-help)
  - [1.6 System Information](#16-system-information)
  - [1.7 Navigation](#17-navigation)
  - [1.8 Working with Files and Directories](#18-working-with-files-and-directories)
  - [1.9 Editing Files](#19-editing-files)
  - [1.10 Find Files and Directories](#110-find-files-and-directories)
  - [1.11 File Descriptors and Redirections](#111-file-descriptors-and-redirections)
  - [1.12 Filter Contents](#112-filter-contents)
  - [1.13 Regular Expressions](#113-regular-expressions)
  - [1.14 Permission Management](#114-permission-management)
  - [1.15 User Management](#115-user-management)
  - [1.16 Package Management](#116-package-management)
  - [1.17 Service and Process Management](#117-service-and-process-management)
  - [1.18 Task Scheduling](#118-task-scheduling)
  - [1.19 Network Services](#119-network-services)
  - [1.20 Working with Web Services](#120-working-with-web-services)
  - [1.21 Backup and Restore](#121-backup-and-restore)
  - [1.22 File System Management](#122-file-system-management)
  - [1.23 Containerization](#123-containerization)
  - [1.24 Network Configuration](#124-network-configuration)
  - [1.25 Remote Desktop Protocols in Linux](#125-remote-desktop-protocols-in-linux)
  - [1.26 Linux Security](#126-linux-security)
  - [1.27 Firewall Setup (+ iptables algorithm)](#127-firewall-setup--iptables-algorithm)
  - [1.28 System Logs](#128-system-logs)
  - [1.29 Solaris](#129-solaris)
  - [1.30 Shortcuts](#130-shortcuts)
- [2. Structured Breakdown (Cheat Sheet)](#2-structured-breakdown-cheat-sheet)
- [3. Feynman Checks](#3-feynman-checks)

---

## 1. Module Digest

### 1.1 Linux Structure

linux is an operating system (OS): software that manages a computer's hardware and mediates between hardware and the programs running on top of it, same job as Windows or macOS. the difference is that linux ships as hundreds of "distributions" (distros), each a different packaging of the same kernel plus different tooling and defaults (Ubuntu, Debian, Fedora, Parrot OS, etc). the interactive labs in this module run on Pwnbox, which is Parrot OS, a Debian based distro built for security work.

**history, briefly:** Unix (Ken Thompson & Dennis Ritchie, AT&T, 1970) → BSD (1977, entangled in an AT&T lawsuit over reused Unix code) → GNU project (Richard Stallman, 1983, aiming for a free Unix like OS, also the source of the GPL license) → Linux kernel (Linus Torvalds, 1991, a personal project that became the missing piece GNU needed). today's kernel is 23M+ lines of C, licensed GPLv2.

**why linux matters for security work specifically:** it's less malware prone than Windows, patches fast, and its "everything is a file" design means you can inspect and control almost anything on the system with the same small set of text based tools. that same transparency is exactly what makes it a good platform for offensive and defensive tooling.

**philosophy, five principles:**

|principle|what it means|
|---|---|
|everything is a file|hardware devices, processes, network connections, all represented as files you can read/write with standard tools|
|small, single purpose programs|each tool does one job well|
|chain programs together|pipe small tools together to build complex behavior|
|avoid captive UIs|the shell is the primary interface, not a locked down GUI|
|config in text files|e.g. `/etc/passwd` stores every registered user as plain text|

_why this matters mechanically_: because config and system state are plain text files rather than a binary registry or opaque database, you can `cat`, `grep`, and `diff` your way through almost the entire system. that's the practical payoff of "everything is a file", not just a slogan.

**components:**

|component|description|
|---|---|
|bootloader|code that starts the boot process (Parrot uses GRUB)|
|OS kernel|manages hardware I/O at the lowest level|
|daemons|background services (scheduling, printing, etc), load after boot/login|
|OS shell|the command line interpreter between user and OS (Bash, Zsh, Fish, etc)|
|graphics server|"X" / X server, lets graphical programs run locally or remotely|
|window manager|the actual GUI (GNOME, KDE, MATE, ...)|
|utilities|the actual programs/tools that do work|

**architecture, layered:** hardware → kernel (virtualizes and arbitrates access to CPU/memory/etc, gives each process its own virtual resources) → shell (CLI to the kernel's functions) → system utility layer (exposes OS functionality to the user).

**filesystem hierarchy standard (FHS), the top level directories:**

|path|contains|
|---|---|
|`/`|root of everything, boot critical files live here before other filesystems mount|
|`/bin`|essential command binaries|
|`/boot`|bootloader, kernel executable, boot files|
|`/dev`|device files (hardware access)|
|`/etc`|local system + application config|
|`/home`|per user storage|
|`/lib`|shared libraries needed at boot|
|`/media`|removable media mount points|
|`/mnt`|temporary mount point for regular filesystems|
|`/opt`|optional/third party software|
|`/root`|root user's home|
|`/sbin`|system admin binaries|
|`/tmp`|temp files, cleared on boot, can vanish anytime|
|`/usr`|executables, libraries, man pages|
|`/var`|variable data: logs, mail, cron, web app files|

Q: why does `/etc/passwd` being plain text matter offensively? A: it (and files like it) can be read, grepped, and diffed with zero special tooling, which is exactly why misconfigured permissions on config files are such a common enumeration target.

---

### 1.2 (not in source material)

not present in `combined.md`. based on the module's numbering this slot likely covers the filesystem itself (as opposed to 1.1's high level structure), but I'm not going to guess at content that wasn't provided.

---

### 1.3 Introduction to Shell

a **shell** (terminal, command line) is the text based I/O interface between you and the kernel. think of the shell as the actual "engine room" and the **terminal** as the front desk you talk to it through, terminal emulation software is what lets that front desk run inside a graphical window instead of on a dedicated physical console. a **CLI** running inside another terminal (like Tmux panes) is just multiple front desks open at once, each talking to the engine room independently.

the default shell on most linux systems is **Bash** (Bourne Again SHell, part of GNU). everything doable through a GUI is doable through Bash, plus scripting for automation. other shells exist: Tcsh/Csh, Ksh, Zsh, Fish. terminal multiplexers (Tmux) let you split one terminal into multiple panes/workspaces.

Q: why do pentesters lean on the shell instead of a GUI file manager? A: it's scriptable, chainable via pipes, and works identically over a bare SSH session where no GUI exists at all.

---

### 1.4 Prompt Description

the bash prompt is a template, not a hardcoded string. by default it shows `<username>@<hostname>[<cwd>]$`, where `~` marks your home directory. the prompt character flips from `$` (unprivileged) to `#` (root) automatically, this is a visual convention, not a security boundary; a script printing `#` doesn't make you root.

the prompt is controlled by the **PS1** environment variable, customized in `.bashrc` using escape sequences:

|char|meaning|
|---|---|
|`\u`|current username|
|`\h`|hostname|
|`\H`|full hostname|
|`\w`|full path of cwd|
|`\d`|date (Mon Feb 6)|
|`\D{%Y-%m-%d}`|date, custom format|
|`\t`|time, 24h (HH:MM:SS)|
|`\T`|time, 12h|
|`\@`|current time|
|`\s`|shell name|
|`\j`|number of jobs managed by the shell|
|`\n`|newline|
|`\r`|carriage return|

for pentesting specifically, a custom PS1 (full path instead of just dirname, target IP baked in, timestamp) makes session logs and `.bash_history` far easier to correlate after the fact, this is exactly why documenting prompt changes matters for maintaining credibility on an engagement.

#### my personal actions

added a timestamp to the default Parrot prompt by inserting `\@` (current time, 12h) right after the hostname escape `\h`:

```bash
export PS1="\[\e]0;\u@\h:\w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\@\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$"
```

breaking down what changed: the original prompt was `...\u@\h\[\033[00m\]:...`, i.e. username `@` hostname, then a color reset, then the cwd. I inserted `\@` between `\h` and the color reset escape, so the visible prompt now reads `user@host<time>:cwd$` instead of just `user@host:cwd$`. per the table above, `\@` renders as the current time, this is the same trick the module describes for tracking actions during an engagement: a plain `user@host` prompt tells you who and where, adding `\@` tells you when, which matters once you're correlating prompt output against `.bash_history` or a captured terminal log.

---

### 1.5 Getting Help

three main ways to get unstuck on an unfamiliar command:

|method|syntax|notes|
|---|---|---|
|man pages|`man <tool>`|full manual, paged (`h` for help, `q` to quit)|
|built in help|`<tool> --help` (or `-h` for some tools, e.g. `curl -h`)|quick option summary, no paging|
|apropos|`apropos <keyword>`|searches every man page's short description for a keyword|

example: `apropos sudo` returns every sudo related man page (`sudo`, `sudoers`, `visudo`, `sudoreplay`, ...) even if you didn't know those tool names existed.

external resource worth bookmarking: explainshell.com, breaks down a full compound command flag by flag.

Q: you don't know if a tool supports `--help` or `-h`, what do you try first? A: `man <tool>` is the safest universal fallback since virtually everything on a standard system ships a man page, even if its own `--help` output is nonexistent or terse.

---

### 1.6 (not in source material)

not present in `combined.md`. your personal log has an entry numbered "section 6" (basic `uname`, `cd`/`pwd`, `env` inspection, `ifconfig` MTU) that would slot in here by the module's own numbering, so it's kept below even without HTB content to pair it with.

#### my personal actions

worked through a short interactive lab covering basic system/environment inspection:

1. `uname -m`: machine hardware name (CPU architecture, e.g. `x86_64`).
2. `cd ~` then `pwd`: confirmed home directory and current working directory.
3. `env`: listed environment variables, looked up `$MAIL` specifically (points at the local mailbox file for the current user, historically `/var/mail/<user>` or similar).
4. `env` again, this time looking up `$SHELL` (path to the user's default login shell, e.g. `/bin/bash`).
5. `uname -s` (found via `uname --help`): prints just the kernel name (`Linux`), as opposed to `uname -a`'s full string.
6. `ifconfig | grep 1500`: filtered `ifconfig` output for the string `1500`, which is the default Ethernet **MTU** (Maximum Transmission Unit): the largest packet size, in bytes, an interface can send in one frame without fragmenting it. grepping for the number is a quick way to spot which interfaces are using non default MTUs (jumbo frames, tunnels, etc), since anything that isn't `1500` on a standard wired interface is worth a second look.

---

### 1.7 Navigation

the core loop: `pwd` (where am I) → `ls` / `ls -l` / `ls -la` (what's here) → `cd` (move).

`ls -l` columns, left to right:

|column example|meaning|
|---|---|
|`drwxr-xr-x`|type + permissions (`d` = directory, `-` = file, `l` = link)|
|`2`|number of hard links|
|`cry0l1t3`|owner|
|`htbacademy`|group|
|`4096`|size in bytes (or block count for a directory)|
|`Nov 13 17:37`|last modified timestamp|
|`Desktop`|name|

hidden files/directories (dotfiles, e.g. `.bashrc`, `.bash_history`) only show up with `ls -la` (`-a` = all, including dotfiles).

`ls` doesn't require you to `cd` somewhere first, you can point it at any path: `ls -l /var/`.

**moving around:**

- `cd <path>`: jump to a directory, absolute or relative.
- `cd -`: jump back to the previous directory (a one entry history).
- `[TAB]` twice: autocomplete, lists every match if there's ambiguity (`cd /dev/s[TAB][TAB]` → `shm/ snd/`).
- inside any directory, `.` is a self reference (the current directory) and `..` is the parent, so `cd ..` always goes up one level.
- `clear` or `[Ctrl] + L`: clear the terminal.
- `[↑]` / `[↓]`: scroll command history; `[Ctrl] + R`: search command history interactively.

#### my personal actions

- `ls -alps`: listed all files (`-a`), long format (`-l`), human unfriendly but exact (`-p` appends `/` to directory names, `-s` shows allocated block size per entry). used this to get a full, unambiguous inventory of a directory including hidden entries and their on disk size.
- `ls -i | grep sudoers`: `-i` prints each file's **inode number** (the filesystem's internal ID for the file, separate from its name), piped through `grep` to isolate the `sudoers` file's inode specifically. this is the same technique the module's own quiz question was built around (finding the inode number of a specific file in `/etc`), inode numbers matter because two different filenames can point at the same inode (hard links), so grepping by inode is a way to track a file's real identity independent of what it's currently named.

---

### 1.8 Working with Files and Directories

the linux equivalent of drag and drop file management, done from the shell:

|command|syntax|does|
|---|---|---|
|`touch`|`touch <name>`|create an empty file (or bump the mtime of an existing one)|
|`mkdir`|`mkdir <name>`|create a directory|
|`mkdir -p`|`mkdir -p a/b/c`|create nested parent directories in one shot, instead of `mkdir a && mkdir a/b && mkdir a/b/c`|
|`tree`|`tree .`|print the full directory structure visually|
|`mv`|`mv <src> <dst>`|move **and** rename (same command does both, the distinction is purely "did the path change directory or just name")|
|`cp`|`cp <src> <dst>`|copy|

worked example from the module, building up a structure step by step:

```bash
touch info.txt
mkdir Storage
mkdir -p Storage/local/user/documents
touch ./Storage/local/user/userinfo.txt
mv info.txt information.txt
touch readme.txt
mv information.txt readme.txt Storage/
cp Storage/readme.txt Storage/local/
```

deletion (`rm`, `rmdir`) wasn't walked through step by step in the source material, it was left as a self directed exercise, the reasoning being that looking things up yourself is part of building real proficiency, not "cheating." for reference: `rm <file>` deletes a file, `rm -r <dir>` recursively deletes a directory and its contents, `rmdir <dir>` only deletes an already empty directory.

Q: why does `mv` handle both "rename" and "move" as literally the same operation? A: because a rename is just a move within the same directory, from the filesystem's perspective there's no meaningful difference, both are "change this path to that path."

#### my personal actions

- `ls -halpsit | head -5`: combined several flags at once: `-h` (human readable sizes), `-a` (all, including dotfiles), `-l` (long format), `-p` (append `/` to dirs), `-s` (block size), `-i` (inode number), `-t` (sort by modification time, newest first), piped through `head -5` to see only the 5 most recently touched entries.
- `ls -halpsit | grep gshadow`: same flag combo, filtered for `gshadow` specifically to pull its inode and size in one line. `/etc/gshadow` stores encrypted group passwords, worth having eyes on since misconfigured permissions there are a known privilege escalation angle.

---

### 1.9 Editing Files

two editors worth knowing: **Nano** (simple, modeless, good for quick edits) and **Vim** (modal, steep learning curve, extremely fast once internalized).

**Nano:** `nano <file>` opens/creates a file directly. the bottom two lines show shortcuts, `^` means `[Ctrl]`. `[Ctrl+W]` searches, `[Ctrl+O]` saves ("write out"), `[Ctrl+X]` exits.

**Vim** is modal: unlike Nano, it distinguishes between "typing is a command" and "typing is text," which is the source of both its power and its reputation for being confusing at first.

|mode|what typing does|
|---|---|
|Normal|every keystroke is a command, not inserted text (default mode on open)|
|Insert|keystrokes are inserted into the buffer as text|
|Visual|select/highlight a block of text to act on|
|Command|single line commands at the bottom (`:` prompt), e.g. save/quit/search/replace|
|Replace|typed text overwrites existing text instead of inserting|
|Ex|multiple commands run sequentially without dropping back to Normal mode between each|

to quit: `:q` (from Command mode, i.e. press `:` then type `q`).

vim ships its own interactive tutorial: `vimtutor` from the shell, or `:Tutor` from inside vim itself. it takes roughly 25 to 30 minutes and is the recommended on ramp, since reading about vim commands without executing them doesn't build the muscle memory.

**why files like `/etc/passwd` matter here:** it stores usernames, UIDs, GIDs, and home directories in plain text. password hashes used to live there too but now live in `/etc/shadow` with much stricter permissions, if `/etc/passwd` or `/etc/shadow` ever have loose permissions, that's a direct privilege escalation opening.

Q: why is Nano's "everything you type is inserted" behavior actually a limitation for power users? A: because every single edit, even a global find/replace, has to go through manual keystrokes with no scriptable command layer, vim's Command mode gives you that layer, which is why vim scales to complex edits far better once you're fluent.

#### my personal actions

worked through `vimtutor` interactively. here's a conspect (a standard `vimtutor` covers this same core set regardless of session, so this is the reference cheat sheet for what those lessons teach, not a verbatim transcript):

|category|keys|does|
|---|---|---|
|movement|`h` `j` `k` `l`|left / down / up / right, one character at a time|
|movement|`w` / `b`|jump forward / backward one word|
|movement|`0` / `$`|jump to start / end of the current line|
|movement|`gg` / `G`|jump to top / bottom of the file|
|deletion|`x`|delete the character under the cursor|
|deletion|`dw`|delete a word|
|deletion|`dd`|delete (cut) the current line|
|deletion|`d$`|delete from cursor to end of line|
|insert|`i`|insert before cursor|
|insert|`a`|insert after cursor (append)|
|insert|`o` / `O`|open a new line below / above and enter insert mode|
|undo/redo|`u`|undo last change|
|undo/redo|`[Ctrl+R]`|redo|
|yank/paste|`yy`|yank (copy) the current line|
|yank/paste|`p`|paste after cursor|
|search|`/pattern`|search forward for `pattern`|
|search|`n` / `N`|repeat search, same / opposite direction|
|replace|`:%s/old/new/g`|replace every `old` with `new`, file wide|
|save/quit|`:w`|write (save)|
|save/quit|`:q` / `:q!`|quit / quit without saving|
|save/quit|`:wq`|save and quit|

the throughline across all of it: normal mode keystrokes are composable little verbs (`d` = delete, `y` = yank) combined with nouns (`w` = word, `$` = end of line), so `dw` reads as "delete word" and `d$` reads as "delete to end of line." that composability is the actual payoff of the modal design, once it clicks, you're not memorizing dozens of isolated shortcuts, you're combining a small verb set with a small noun set.

---

### 1.10 Find Files and Directories

three tools, three different jobs, easy to conflate at first:

|tool|answers|speed|how|
|---|---|---|---|
|`which`|"where is the binary for this command"|instant|checks `$PATH` only|
|`locate`|"where is any file matching this name"|instant|queries a prebuilt index/database|
|`find`|"where is any file matching these criteria"|slower|walks the actual filesystem live, right now|

`which` example: `which python3` tells you if/where a binary is available on `$PATH`, useful for checking what tools (curl, wget, python, gcc, nc) exist on a target before you rely on them.

`find` is the heavyweight, it supports arbitrary filters combined together: name pattern, size, modification time, permissions, and more, then an action to run on every match.

```bash
find / -name '*.conf' -size +25k ! -size +28k -newermt 2020-03-03 2>/dev/null -exec ls -halFsit {} \;
```

reading that left to right: search from `/`, name matches `*.conf`, size over 25k but not (`!`) over 28k, modified after 2020-03-03, silence permission errors (`2>/dev/null`), then run `ls -halFsit` on every match (`{}` is replaced by the matched path, `\;` terminates the `-exec` clause).

`locate`'s catch: it reads from a database (`updatedb`), not the live filesystem, so a file created seconds ago won't show up until that database refreshes, which is exactly the failure mode described below.

Q: `find /etc/ -name shadow` fails with "Permission denied" printed alongside the actual result, what's happening and how do you clean it up? A: `find` is printing STDERR (the permission errors) interleaved with STDOUT (the actual matches) on the same terminal, redirect STDERR to `/dev/null` (`2>/dev/null`) to see only real results.

#### my personal actions

1. `find / -name '*.conf' -size +25k ! -size +28k -newermt 2020-03-03 2>/dev/null -exec ls -halFsit {} \;`: as broken down above.
2. `find / -name '*.bak' 2>/dev/null | nl`: found every `.bak` file system wide, piped through `nl` to number each line (a quick way to eyeball how many results came back without a separate `wc -l`).
3. `which xxd`: confirmed the `xxd` hex dump utility is available and located its path.

also (logged separately but same topic): tried `locate '.log' | wc -l` first and got an incorrect count, most likely because `updatedb`'s index hadn't refreshed since the relevant files were created, this is exactly the "database vs live filesystem" gap in the table above. switched to `find / -name '*.log' -type -f 2>/dev/null | wc -l` and got the correct count, since `find` always walks the live filesystem rather than a cached index.

---

### 1.11 File Descriptors and Redirections

a **file descriptor (FD)** is a kernel maintained reference number for an open file, socket, or other I/O resource, the mental model the module uses is a coat check ticket: the ticket (FD) isn't the coat (the actual resource), it's how the system knows which resource you mean when you hand it over.

the first three FDs exist by default on every process:

|FD|name|direction|
|---|---|---|
|0|STDIN|input|
|1|STDOUT|output|
|2|STDERR|output, specifically for errors|

**redirection operators:**

|operator|does|
|---|---|
|`>`|redirect STDOUT to a file, overwrite if it exists|
|`>>`|redirect STDOUT to a file, append instead of overwrite|
|`2>`|redirect STDERR specifically|
|`2>/dev/null`|discard STDERR entirely (`/dev/null` is a black hole device)|
|`<`|redirect a file's contents into STDIN|
|`<<EOF`|heredoc: stream input until the literal `EOF` marker|
|`\|`|pipe: feed one command's STDOUT into the next command's STDIN|

worked progression from the module:

```bash
find /etc/ -name shadow                          # errors and results interleaved
find /etc/ -name shadow 2>/dev/null               # errors silenced
find /etc/ -name shadow 2>/dev/null > results.txt # STDOUT only, saved to file
find /etc/ -name shadow 2> stderr.txt 1> stdout.txt  # STDOUT and STDERR to separate files
cat < stdout.txt                                  # feed a file in as STDIN
find /etc/ -name passwd >> stdout.txt 2>/dev/null # append instead of overwrite
find /etc/ -name *.conf 2>/dev/null | grep systemd            # pipe into grep
find /etc/ -name *.conf 2>/dev/null | grep systemd | wc -l    # chain further into wc
```

_why the numbers matter_: `2>` isn't arbitrary syntax, `2` is literally FD 2 (STDERR). `>` with no number defaults to FD 1 (STDOUT) because that's the conventional default target, this is exactly why `2>/dev/null` silences errors specifically while regular output still prints normally.

Q: what's the actual difference between `>` and `>>`? A: `>` truncates (overwrites) the target file if it already exists, `>>` appends to the end, same target, opposite behavior on collision.

#### my personal actions

- explored `locate` and `dpkg -l | grep ii | wc -l` as part of this section's practice: the `dpkg -l` command lists every package known to dpkg with a two letter status code per line, `ii` specifically means "installed and configured correctly" (as opposed to `rc` = removed but config remains, or `un` = never installed). piping through `grep ii | wc -l` counts only the packages that are actually fully installed right now, filtering out half installed or removed cruft that would otherwise inflate a naive line count.

---

### 1.12 (not in source material)

not present in `combined.md`, but your log has an entry numbered "section 12" that would slot in here by the module's own numbering, kept below even without HTB content to pair it with.

#### my personal actions

- `ss -tnlp4 | grep -E '0.0.0.0:[0-9]+' | wc -l` — `ss` is the modern replacement for `netstat`, `-t` filters to TCP, `-n` skips DNS resolution (numeric output only, faster and avoids leaking lookups), `-l` shows only listening sockets, `-4` restricts to IPv4. piping through `grep -E '0.0.0.0:[0-9]+'` isolates sockets bound to all interfaces (`0.0.0.0`, as opposed to a single specific IP), then `wc -l` counts them, a quick way to get "how many services are listening on every interface" as a single number.
- first tried `ss -tlp | grep -i proftp` to find the ProFTPD process, came back empty, switched to `ps aux | grep -i proftp` instead and found it that way. worth flagging as a lesson: `ss -tlp` only shows sockets that are actually listening right now, if the service is running but not currently bound the way you expect (or the grep pattern doesn't match how `ss` labels it), `ps aux` is the more reliable fallback since it searches process names/command lines directly rather than socket state.
- back on the Pwnbox side, ran a one liner to enumerate every unique link on a website:
    
    ```bash
    curl -s https://www.inlanefreight.com | tr " " "\n" | tr "'" '"' | grep -E '"(https?://www.inlanefreight\.com)?/[^/][^"]*' | cut -d '"' -f 2 | sort -u | wc -l
    ```
    
    reading it left to right: `curl -s` fetches the page quietly (no progress meter), the two `tr` calls split the HTML on spaces (one "token" per line) and normalize single quotes to double quotes so the next step has one consistent quote style to match against, `grep -E` pulls out anything that looks like a link (either a bare `/path` or a full `https://www.inlanefreight.com/path`) still wrapped in its surrounding quote character, `cut -d '"' -f 2` strips the quotes off to leave just the path/URL itself, `sort -u` deduplicates, `wc -l` gives the final count. a crude but genuinely effective way to enumerate a site's internal links without a real crawler.

---

### 1.13 Regular Expressions

regex is a pattern language for matching text, available across most linux tools (`grep`, `sed`, `awk`, etc). a regex is built from literal characters plus **metacharacters** that describe structure rather than literal text.

**grouping operators:**

|operator|example|does|
|---|---|---|
|`()`|`(a)`|groups a subpattern|
|`[]`|`[a-z]`|character class, matches any one char in the set|
|`{}`|`{1,10}`|quantifier, repeat count or range|
|`\|`|`(my\|false)`|OR, matches either side|
|`.*`|`(my.*false)`|effectively an AND when both terms must appear in order|

`-E` enables extended regex in `grep` (needed for the operators above to work as shown, without it some of this syntax needs escaping or won't be recognized).

```bash
grep -E "(my|false)" /etc/passwd     # OR: lines containing "my" OR "false"
grep -E "(my.*false)" /etc/passwd    # AND: lines containing "my" AND "false", in that order
grep -E "my" /etc/passwd | grep -E "false"   # same AND result, done as two separate greps
```

_why `.*` acts like an AND here_: it's not actually a boolean AND operator, it's "match `my`, then anything, then `false`," which only succeeds if both substrings exist in that order on the same line. a true unordered AND (both terms present regardless of order) is what chaining two separate `grep` calls achieves instead.

Q: why does `grep -E "(my.*false)"` fail to match a line where `false` appears before `my`? A: `.*` only matches characters that come after `my` in the string, the pattern is inherently order sensitive, it's shorthand for a specific sequence, not a general "both present" test.

#### my personal actions

worked through the module's six optional regex exercises against `/etc/ssh/sshd_config`:

1. `grep -v '#'`: `-v` inverts the match, so this shows every line that does **not** contain `#` (i.e. strips comments).
2. `grep -E '^Permit'`: `^` anchors to line start, matches lines where a word starts with `Permit`.
3. `grep -E 'Authentication$'`: `$` anchors to line end, matches lines ending in `Authentication`.
4. `grep 'Key'`: plain substring match for lines containing `Key`.
5. `grep -E '^Password.*yes'`: combines a start anchor with the AND pattern from above: lines starting with `Password` that also contain `yes` somewhere after it.
6. `grep -E 'yes$'`: lines ending in `yes`.

---

### 1.14 Permission Management

every file/directory has an **owner**, a **group**, and a permission set for three actors: **owner (u)**, **group (g)**, **others/world (o)**. three permission types apply to each actor: **read (r)**, **write (w)**, **execute (x)**.

|symbol|file meaning|directory meaning|
|---|---|---|
|`r`|view file contents|list directory contents|
|`w`|modify file contents|create/delete entries inside the directory|
|`x`|execute the file as a program/script|`cd` into the directory|

`ls -l` shows this as a 10 character string, e.g. `-rwxr-xr--`: position 1 is the type (`-` file, `d` directory, `l` symlink), then three groups of `rwx` for owner/group/other in that order.

**octal notation** is the same information as a number: `r=4, w=2, x=1`, summed per actor. `rwxr-xr--` → owner `4+2+1=7`, group `4+0+1=5`, other `4+0+0=4` → `754`. this works because each bit position is a distinct power of 2 within a 3 bit field, so any combination of r/w/x maps to exactly one number 0 to 7 with no overlap, that's why octal can losslessly represent the same 9 bit permission string in 3 digits.

**changing permissions:**

|command|example|does|
|---|---|---|
|`chmod`|`chmod 754 file`|set exact permissions via octal|
|`chmod`|`chmod u+x file`|symbolic mode: add execute for owner only|
|`chown`|`chown user:group file`|change owner and/or group|

**special permissions**, beyond the basic 9 bits:

|bit|on a file|on a directory|
|---|---|---|
|SUID (`s` in owner's `x` slot)|runs with the **owner's** privileges, not the caller's|no effect|
|SGID (`s` in group's `x` slot)|runs with the **group's** privileges|new files inside inherit the directory's group|
|sticky bit (`t` in other's `x` slot)|no effect|only the file's owner (or root) can delete/rename files inside, even if others have write access|

_why SUID matters for security specifically_: a SUID binary owned by root runs with root's privileges no matter who executes it, that's exactly why misconfigured or unnecessary SUID binaries are one of the most common linux privesc vectors, `find / -perm -4000 2>/dev/null` (searching for the SUID bit) is a standard first move during enumeration.

Q: a file shows `rwsr-xr-x`, what does the `s` tell you and why should it raise a flag during enumeration? A: it's the SUID bit set in the owner's execute slot, meaning the binary runs as its owner (often root) regardless of who invokes it, if that owner is root and the binary lets you break out to a shell or read/write arbitrary files, it's a direct path to privilege escalation.

---

### 1.15 User Management

three account types on a linux system:

|type|purpose|
|---|---|
|root|full unrestricted access, UID 0|
|system/service accounts|run specific services (e.g. `www-data`, `mysql`), not meant for interactive login|
|regular users|normal login accounts with restricted privileges|

**user info lives in:**

|file|contains|
|---|---|
|`/etc/passwd`|username, UID, GID, home dir, login shell, plain text, world readable|
|`/etc/shadow`|password hashes, tightly restricted permissions|
|`/etc/group`|group definitions and membership|

**core commands:**

|command|does|
|---|---|
|`useradd -m -s /bin/bash -G sudo -g primarygroup devs`|create user `devs`, `-m` makes a home directory, `-s` sets login shell, `-G` sets supplementary groups, `-g` sets primary group|
|`usermod`|modify an existing user (shell, groups, lock status, etc)|
|`usermod --lock <user>`|lock an account (disables password login without deleting it)|
|`userdel`|delete a user|
|`passwd`|set/change a password|
|`su <user>`|switch to another user (prompts for their password)|
|`sudo <cmd>`|run a single command as another user (root by default), governed by `/etc/sudoers`|

Q: why does `useradd` need `-m` explicitly instead of always creating a home directory? A: not every account needs one, service accounts especially, forcing it to be explicit avoids silently cluttering `/home` with directories for accounts that will never log in interactively.

#### my personal actions

- `useradd -m -s /bin/bash -G sudo devs -g primarygroup123`: created a new user `devs` with a home directory (`-m`), bash as login shell (`-s`), `sudo` as a supplementary group (`-G`), and `primarygroup123` as the primary group (`-g`). kept this exact command as the one to memorize going forward, since it covers the four things that actually matter when spinning up a new interactive user: shell, home dir, primary group, and supplementary group.
- `usermod --lock`: locked an account without deleting it, useful for disabling access while preserving the account's ownership records on files it created.
- `su root --command "cat /etc/shadow"`: switched to root just long enough to run a single command (`cat /etc/shadow`), rather than dropping into a full root shell. `--command` runs the given command as root and returns immediately to the original shell, a smaller footprint than a full `su -` session.

---

### 1.16 Package Management

three layers, each wrapping the one below it:

|layer|tool (Debian family)|scope|
|---|---|---|
|low level|`dpkg`|installs/removes a single `.deb` package file directly, no dependency resolution|
|high level|`apt` / `apt-get`|resolves dependencies, fetches from configured repositories, wraps `dpkg`|
|GUI|e.g. Synaptic|frontend over `apt`|

**common commands:**

|command|does|
|---|---|
|`apt update`|refresh the local package index from repositories, doesn't install anything by itself|
|`apt upgrade`|actually upgrade installed packages to their newest available version|
|`apt install <pkg>`|install a package (and its dependencies)|
|`apt remove <pkg>`|remove a package, config files may remain|
|`apt purge <pkg>`|remove a package **and** its config files|
|`apt-cache search <term>`|search package names/descriptions for a keyword|
|`apt-cache show <pkg>`|show metadata about a package (version, dependencies, description)|
|`apt list --installed`|list every currently installed package|
|`dpkg -l`|list packages dpkg knows about, with status codes (`ii` = installed)|
|`dpkg -i <file.deb>`|install a local `.deb` file directly|

_why `update` and `upgrade` are two separate steps_: `update` only refreshes the index (what's available and at what version), it changes nothing on disk. `upgrade` is the step that actually installs newer versions, keeping them separate means you can see what would change before committing to it.

Q: you want to check whether a tool like `impacket` is packaged for this distro before trying to install it from source, what do you run? A: `apt-cache search impacket` (or `apt-cache show impacket` if you already know the exact package name and want its details).

---

### 1.17 Service and Process Management

a **service** (daemon) is a background process, usually started at boot and managed by the init system. modern linux distros mostly use **systemd**, controlled through `systemctl`.

**systemctl basics:**

|command|does|
|---|---|
|`systemctl status <service>`|show current state (running/stopped/failed) plus recent log lines|
|`systemctl start/stop/restart <service>`|control a service right now|
|`systemctl enable/disable <service>`|control whether it starts automatically at boot|
|`systemctl list-units`|list all currently loaded units (services, sockets, targets, etc)|
|`systemctl list-units --type=service`|filter to services only|

`systemctl list-units` output can be searched interactively by pressing `/` then typing a search term (it's paged through `less` under the hood), useful for locating a specific unit by partial name without scrolling manually.

**process inspection:**

|command|does|
|---|---|
|`ps aux`|snapshot of every running process, owner, CPU/mem usage, command|
|`top` / `htop`|live updating process view, `htop` is the friendlier, more visual version|
|`kill <PID>`|send a termination signal to a specific process by ID|
|`killall <name>`|kill every process matching a name|

_why `ps aux` over just `ps`_: bare `ps` only shows processes attached to your current terminal session, `aux` (`a` = all users, `u` = user oriented output, `x` = include processes with no controlling terminal, e.g. daemons) gives you the full system wide picture, which is what you actually want when hunting for a specific running service.

Q: `systemctl status sshd` shows `active (running)` but you can't connect, what's the next thing to check? A: not the service state itself, since that's already confirmed healthy, check what it's actually listening on and where (`ss -tlnp | grep ssh` or similar) since the process can be alive while bound to the wrong interface/port, or blocked by a firewall rule downstream.

#### my personal actions

- `systemctl --help`, then `systemctl list-units`, then pressed `/` and searched for "profiles managed" inside the paged output to find the relevant unit without scrolling manually. this is the fast way to locate one specific unit out of a long `list-units` dump: let `less`'s own search do the filtering instead of eyeballing the list.

---

### 1.18 Task Scheduling

**cron** is the standard linux job scheduler, jobs are defined in a **crontab** (cron table), one line per job, five time fields plus the command:

```
* * * * * <command>
│ │ │ │ │
│ │ │ │ └── day of week (0-6, Sunday=0)
│ │ │ └──── month (1-12)
│ │ └────── day of month (1-31)
│ └──────── hour (0-23)
└────────── minute (0-59)
```

a bare `*` in any field means "every value", so `* * * * *` fires every single minute.

**commands:**

|command|does|
|---|---|
|`crontab -e`|edit the current user's crontab|
|`crontab -l`|list the current user's crontab entries|
|`/etc/cron.d/`, `/etc/crontab`|system wide cron jobs, can specify which user runs them|

related but distinct: **systemd timers** (`.timer` unit files) are the systemd native alternative to cron, and **at** (`at <time>`) schedules a job to run exactly once at a given time, rather than on a recurring schedule.

Q: what's the actual difference between a systemd timer and a plain cron job? A: a timer is a systemd unit like any service, so it gets systemd's dependency handling, logging via `journalctl`, and the ability to trigger relative to events (e.g. "5 minutes after boot") rather than only wall clock time, cron is simpler but purely wall clock/calendar based.

#### my personal actions

- `locate dconf.service`, then `cat /usr/share/dbus-1/services/ca.desrt.dconf.service`: tracked down the dconf service's D-Bus activation file to inspect how it's launched. dconf itself is the GNOME configuration database backend, its service file shows it's D-Bus activated (`Type=dbus`) rather than started directly by systemd at boot, meaning it only spins up on demand when something requests it over D-Bus, which explains why it doesn't show up as "running" until something actually touches GNOME's config store.

---

### 1.19 Network Services

a network service is any daemon that listens on a port and talks to other hosts. the module walks through several core ones:

**SSH (Secure SHell)**, port 22: encrypted remote shell access. config at `/etc/ssh/sshd_config` (server side) and `~/.ssh/config` (client side). key based auth (public key on the server in `~/.ssh/authorized_keys`, private key held by the client) is preferred over password auth since it's not brute forceable in any practical sense and can be set up passphrase free for automation.

**NFS (Network File System)**: lets a linux host export part of its filesystem so other hosts can mount it as if it were local. server side config lives in `/etc/exports`, one line per shared directory:

```
/path/to/share  client_ip(option1,option2,...)
```

|option|meaning|
|---|---|
|`rw`|clients can read and write|
|`ro`|clients can only read|
|`sync`|writes are committed to disk before the server acknowledges them (safer, slower)|
|`async`|server acknowledges writes before they're actually committed (faster, riskier on crash)|
|`no_root_squash`|a remote root user keeps root privileges on the share, dangerous by default|
|`root_squash`|(the safe default) a remote root user is mapped down to an unprivileged user (`nobody`) on the share|
|`all_squash`|every remote user, root or not, is mapped to `nobody`|

_why `no_root_squash` is the one to watch for_: NFS by default assumes the client's own OS is enforcing user separation, so a remote root account normally gets squashed down to `nobody` on the share to prevent a compromised or malicious client from writing files as root on the server. `no_root_squash` disables that safety net, if you can become root on any client permitted to mount a share with that option set, you can write arbitrary files (including SUID binaries) as root on the NFS server itself, this is a textbook privilege escalation / lateral movement vector and one of the first things worth checking (`showmount -e <target>`, then inspect `/etc/exports` if you get local access) during a linux focused engagement.

**other services covered:** DNS (name resolution), DHCP (automatic IP assignment), FTP (file transfer, cleartext), NTP (time sync). each is a background daemon with its own config file under `/etc/` and its own well known port.

Q: you find a mount option of `rw,no_root_squash` on a share you can reach, why is that specifically more dangerous than just `rw`? A: `rw` alone still squashes a remote root user down to `nobody`, so even with write access you can't write files owned by root, adding `no_root_squash` removes that squash, meaning root on the client really is root on the share, letting you plant a root owned SUID binary the server itself will execute with full privileges.

#### my personal actions

logged the full `/etc/exports` option table above verbatim as a reference, flagged as foundational knowledge worth having memorized rather than looked up mid engagement, specifically because `no_root_squash` is easy to miss during a quick skim of an exports file but is one of the highest value findings if present.

---

### 1.20 Working with Web Services

two ways to spin up a throwaway HTTP server for quick file transfers or testing, both covered in the module:

**python's built in server:**

```bash
python3 -m http.server 8000
```

no install needed if python3 is already present, serves the current directory over HTTP on the given port.

**PHP's built in server:**

```bash
php -S 127.0.0.1:8080
```

`-S` starts PHP's built in dev server bound to the given address:port, also serves and can execute `.php` files in the current directory, not just static files like the python option.

Q: why would you reach for the PHP server over the python one specifically? A: if the files you need to serve/test include actual `.php` scripts that need to execute, python's `http.server` only serves static files, it won't run PHP code, `php -S` will.

#### my personal actions

- ran `npm --help` first without knowing what npm actually was, googled it (Node Package Manager, the package manager bundled with Node.js), then googled how to launch an HTTP server via npm and how to specify a port, landed on:
    
    ```bash
    npm install http-servernpx http-server -p 8080
    ```
    
    `npm install http-server` installs the `http-server` package locally, `npx http-server -p 8080` runs it without needing a separate global install step, `-p 8080` sets the listening port.
- separately, ran `php -S 127.0.0.1:8080` after finding the flag via `php --help | grep 'server'`, matching the module's own example above.

---

### 1.21 Backup and Restore

three tools for moving/preserving data, escalating in capability:

|tool|does|
|---|---|
|`cp -r`|simple recursive copy, no sync intelligence|
|`rsync`|sync tool, only transfers the differences between source and destination, can run locally or over SSH|
|`tar`|bundles many files into a single archive (optionally compressed), for full backups/snapshots|

**rsync core flags:**

|flag|meaning|
|---|---|
|`-a`|archive mode: recursive, preserves permissions/timestamps/symlinks/ownership, bundles several sensible defaults into one flag|
|`-v`|verbose|
|`-z`|compress data during transfer|
|`-e "ssh ..."`|tunnel the sync over SSH instead of a plain, unencrypted connection|

`crontab -e` + a scheduled `rsync` call is the standard pattern for automated, unattended backups.

Q: why does `-a` matter more than just `-r` (recursive) alone? A: `-r` only handles descending into subdirectories, `-a` additionally preserves permissions, ownership, and timestamps, without it a "backup" silently loses metadata that matters (e.g. a restored SSH key with the wrong permissions will be rejected outright by `sshd`).

#### my personal actions

built the module's full optional lab end to end, an automated, passwordless, SSH backed backup running on a schedule:

**1. mock data:**

```bash
mkdir -p ~/to_back ~/to_sync
echo "very important secret" > ~/to_back/text.txt
```

**2. passwordless SSH setup**, so the cron job never has to prompt for a password:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# generate a keypair with no passphrase, dedicated to this one task
ssh-keygen -t rsa -b 2048 -f ~/key -N ""

# authorize that key for local login
cat ~/key.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# confirm it connects with zero prompts
ssh -i ~/key -o StrictHostKeyChecking=no $USER@127.0.0.1 "echo 'Manual key installed'"
```

`chmod 700 ~/.ssh` and `chmod 600 authorized_keys` aren't cosmetic, `sshd` refuses to use a key or `authorized_keys` file with group/world write permissions, so getting the permissions right up front is what actually makes the passwordless step work at all. `-N ""` sets an empty passphrase (needed since cron can't type one in later), `-o StrictHostKeyChecking=no` skips the interactive host key confirmation prompt, again because nothing will be there to answer it once this runs unattended.

**3. the backup script**, `~/backuper.sh`:

```bash
#!/bin/bash
rsync -avz -e "ssh -i /home/$USER/key -o StrictHostKeyChecking=no" /home/$USER/to_back/ $USER@127.0.0.1:/home/$USER/to_sync/
```

```bash
chmod 700 ~/backuper.sh
```

`-avz`: archive mode + verbose + compressed. `-e "ssh -i ... -o StrictHostKeyChecking=no"` tells rsync to tunnel over SSH using the dedicated key from step 2, rather than a plain connection. `chmod 700` makes the script executable by its owner only.

**4. schedule it**, `crontab -e`:

```
* * * * * /home/$USER/backuper.sh
```

runs every minute, tight enough for testing, would normally be something much less frequent (e.g. `0 2 * * *` for once a day at 2am) in a real deployment.

**5. test:**

```bash
echo "gwarfg" >> ~/to_back/text.txt
```

then waited a minute and confirmed the change showed up in `~/to_sync/`, closing the loop: edit source → cron fires → rsync pushes the diff over the passwordless SSH connection → destination updates, with zero manual intervention after setup.

---

### 1.22 File System Management

**disk/partition inspection:**

|command|shows|
|---|---|
|`lsblk`|block devices as a tree: disks → partitions, sizes, mount points|
|`fdisk -l`|detailed partition table info per disk (needs root)|
|`df -h`|disk space usage per mounted filesystem, human readable|
|`du -sh <path>`|actual space used by a specific file/directory tree|
|`mount`|currently mounted filesystems|
|`/etc/fstab`|filesystems mounted automatically at boot|

**mounting:**

```bash
mount /dev/sdb1 /mnt/usb   # attach a filesystem to a directory (mount point)
umount /mnt/usb            # detach it
```

a filesystem has to be mounted onto an existing directory before its contents become accessible through the normal directory tree, the device itself isn't browsable until it's attached somewhere.

**filesystem types** covered: ext4 (linux's common default), NTFS/FAT32 (windows interop), XFS, and others, each with different journaling/performance/compatibility tradeoffs.

Q: `df -h` shows a partition at 100% but `du -sh` on its contents adds up to far less, what's a likely explanation? A: a common one is deleted files still held open by a running process, the space isn't freed until every process holding the file descriptor closes it, `du` only sees what's currently linked in the directory tree, `df` reports actual block usage including those phantom "deleted but still open" files.

#### my personal actions

- `sudo fdisk -l` (and, as an alternative, `lsblk`): inventoried attached disks and their partition tables. `fdisk -l` needs root since it reads raw block devices directly, `lsblk` is the friendlier, non destructive tree view and doesn't require elevated privileges to read basic layout info.

---

### 1.23 Containerization

**virtualization vs containerization**, the core distinction:

||virtual machine|container|
|---|---|---|
|what's virtualized|full hardware, via a hypervisor|just the OS's process/resource view, via the kernel|
|includes its own kernel|yes|no, shares the host's kernel|
|isolation mechanism|hardware level|kernel namespaces + cgroups|
|overhead|heavier (boots a full OS)|lighter (just a process with a fenced off view)|

_why containers are lighter, mechanically_: a VM emulates hardware and boots an entire independent kernel on top of it, a container is just a normal linux process that the host kernel restricts using two features: **namespaces** (each container gets its own isolated view of PIDs, network interfaces, mounts, hostname, etc, so it _looks_ like a separate machine from the inside) and **cgroups** (control groups, which cap and account for how much CPU/memory/IO that process is allowed to consume). no second kernel is ever booted, that's the entire source of the overhead difference.

**Docker** is the dominant container runtime/tooling:

|concept|what it is|
|---|---|
|image|a read only template (filesystem + metadata) a container is instantiated from|
|container|a running (or stopped) instance of an image|
|Dockerfile|a text recipe describing how to build an image, layer by layer|
|registry|a place images are stored/pulled from (Docker Hub by default)|

**core commands:**

|command|does|
|---|---|
|`docker pull <image>`|download an image from a registry|
|`docker run <image>`|create and start a container from an image|
|`docker run -it <image> bash`|run interactively with a TTY attached, dropping into a shell|
|`docker ps`|list running containers|
|`docker ps -a`|list all containers, including stopped ones|
|`docker images`|list locally available images|
|`docker build -t <name> .`|build an image from a Dockerfile in the current directory|
|`docker exec -it <container> bash`|get a shell inside an already running container|
|`docker stop` / `docker rm`|stop / delete a container|

_why this matters for security work_: a container escape (breaking out of the namespace/cgroup jail back to the host) is a real, well documented attack class, exactly because the isolation is a kernel level software boundary rather than a hardware one, that's structurally weaker than a VM's hypervisor boundary. misconfigurations like running a container with `--privileged` or mounting the host's docker socket into a container are common enumeration targets for that reason.

Q: why is a container escape a meaningfully different threat than a VM escape? A: a container shares the host's actual kernel, so an escape only has to defeat software level isolation (namespaces/cgroups) rather than a hardware enforced hypervisor boundary, that's a structurally smaller barrier, which is why container escapes are both more common and generally lower complexity than VM escapes.

---

### 1.24 Network Configuration

**interface inspection and control:**

|command|does|
|---|---|
|`ip a` (or `ip addr`)|show all interfaces and their assigned IPs, modern replacement for `ifconfig`|
|`ip link`|show/control interfaces at the link (layer 2) level|
|`ip route` (or `route -n`)|show the routing table|
|`ifconfig`|legacy equivalent of `ip a`, still common but deprecated in favor of `ip`|

**bringing an interface up/down:**

```bash
ip link set eth0 up
ip link set eth0 down
```

**assigning an address manually:**

```bash
ip addr add 192.168.1.50/24 dev eth0
```

**DNS**: client side resolution config lives in `/etc/resolv.conf` (nameserver entries) and `/etc/hosts` (static hostname → IP mappings, checked before DNS).

**persistent config** (survives reboot) is handled differently per distro/network manager: `/etc/network/interfaces` (older Debian style), Netplan (`/etc/netplan/*.yaml`, modern Ubuntu), NetworkManager (`nmcli`), or systemd-networkd.

_why `/etc/hosts` gets checked before DNS_: it's the original, purely local override mechanism predating DNS, the resolution order (`/etc/hosts` first, DNS second, by default via `/etc/nsswitch.conf`) means an entry in `/etc/hosts` can silently override what DNS would otherwise return, this is exactly the mechanism behind local DNS spoofing for testing, and also a common thing to check if a hostname is resolving somewhere unexpected.

Q: `ip addr add` sets an address but it doesn't survive a reboot, why not, and what's the fix? A: `ip` commands change the running kernel state directly and don't touch any persistent config file, to survive a reboot the same address needs to be written into whatever this distro's persistent config mechanism is (`/etc/network/interfaces`, a netplan yaml, etc), the live command and the on disk config are two entirely separate things.

---

### 1.25 Remote Desktop Protocols in Linux

three graphical remote access protocols, different tradeoffs:

|protocol|port(default)|notes|
|---|---|---|
|VNC (Virtual Network Computing)|5900+|platform agnostic, sends raw framebuffer pixel data, generally unencrypted unless tunneled|
|RDP (Remote Desktop Protocol)|3389|Microsoft's protocol, natively windows, usable on linux via `xrdp` (server) and clients like `rdesktop`/`Remmina`|
|XRDP|3389|an open source RDP server implementation that lets linux accept RDP connections, bridges linux desktops into the windows native RDP ecosystem|

_why VNC being unencrypted by default matters_: raw VNC traffic (screen data plus keystrokes/mouse events) can be sniffed on the wire, in practice VNC is usually tunneled through SSH (`ssh -L`) for that reason, rather than exposed directly, this is a good example of "the tool doesn't secure itself, the deployment has to."

Q: why would you specifically set up `xrdp` on a linux box instead of just using VNC? A: to let windows admins/clients connect using their native RDP client without needing separate VNC software, `xrdp` translates RDP protocol traffic to linux's own display system, it's a compatibility bridge rather than a linux native protocol.

---

### 1.26 Linux Security

a survey of the major linux hardening/security mechanisms:

|mechanism|protects against|
|---|---|
|firewall (iptables/nftables/ufw)|unwanted inbound/outbound network traffic|
|SELinux / AppArmor|mandatory access control, confines what a process can do even beyond standard unix permissions|
|fail2ban|brute force login attempts, bans IPs after repeated failures|
|auditd|detailed system call level auditing/logging|
|updates/patching|known, already disclosed vulnerabilities|
|least privilege (proper user/group/permission design)|lateral movement and privilege escalation after an initial foothold|

**SELinux vs AppArmor**, both are mandatory access control (MAC) systems layered on top of standard unix discretionary access control (DAC, i.e. the regular owner/group/permission model): DAC lets the file's owner decide who can access it, MAC adds a system wide policy that can restrict access even further, regardless of what the owner has permitted. SELinux (used by RHEL/Fedora family) applies **labels** to every file/process and enforces policy based on those labels, AppArmor (used by Debian/Ubuntu family) is path based, confining programs to explicit lists of what files/capabilities each profile allows.

_why MAC matters on top of already having permissions_: normal permissions are entirely at the discretion of the file's owner, if a process is compromised and running as a user who legitimately owns sensitive files, DAC alone provides zero additional resistance, MAC adds a policy layer the owner can't simply override, which is why a correctly configured SELinux/AppArmor policy can contain a compromised service even when the attacker already has valid access to that service's own user account.

Q: a service running under a strict AppArmor profile gets exploited, why might the attacker still be unable to read `/etc/shadow` even if the service's user technically has no restriction against it in standard permissions? A: because AppArmor is a second, independent policy layer, on top of DAC, if the profile doesn't explicitly permit that process to touch `/etc/shadow`, it's blocked regardless of what the underlying unix permissions would otherwise allow, that's the entire point of confinement operating "in addition to," not "instead of," standard permissions.

---

### 1.27 Firewall Setup (+ iptables algorithm)

a firewall's job is controlling and monitoring traffic between network segments (internal vs external, or between zones), filtering based on rules, protocols, ports, and other criteria. on linux this is implemented via **Netfilter**, a set of hooks built into the kernel that can intercept and modify packets as they pass through, **iptables** is the classic userspace tool for configuring those hooks (introduced in kernel 2.4, 2000, replacing the older `ipchains`/`ipfwadm`). newer alternatives exist: **nftables** (modern syntax, better performance, not rule compatible with iptables), **ufw** ("uncomplicated firewall," a simpler frontend built on top of iptables), and **firewalld** (dynamic zone based management).

**the four building blocks:**

|component|role|
|---|---|
|tables|top level categories of rules, grouped by what kind of work they do|
|chains|within a table, a named hook point traffic passes through|
|rules|criteria + a target, "if this matches, do that"|
|targets|the action taken on a match (accept, drop, modify, log, ...)|

**tables:**

|table|purpose|built in chains|
|---|---|---|
|`filter`|accept/drop/reject decisions based on IP, port, protocol|INPUT, OUTPUT, FORWARD|
|`nat`|rewrite source/destination IPs|PREROUTING, POSTROUTING|
|`mangle`|modify packet header fields|PREROUTING, OUTPUT, INPUT, FORWARD, POSTROUTING|
|`raw`|special case packet processing, runs before connection tracking|PREROUTING, OUTPUT|

**chains**, what each one means:

- `INPUT`: traffic destined for this machine itself.
- `OUTPUT`: traffic generated by this machine itself.
- `FORWARD`: traffic passing through this machine to somewhere else (routing/gateway behavior).
- `PREROUTING`: runs before the kernel decides whether a packet is for this machine or being forwarded.
- `POSTROUTING`: runs after that decision, right before the packet leaves.

chains can also be user defined: a custom named chain you create to group related rules (e.g. every rule for a set of web servers, or everything targeting port 80), then jump into from a built in chain, mainly for organization at scale.

**targets:**

|target|does|
|---|---|
|`ACCEPT`|let the packet through|
|`DROP`|silently discard it|
|`REJECT`|discard it and notify the sender (e.g. ICMP port unreachable)|
|`LOG`|log the packet, non terminating (evaluation continues to the next rule)|
|`SNAT`|rewrite the source IP (used for outbound NAT)|
|`DNAT`|rewrite the destination IP (used for port forwarding)|
|`MASQUERADE`|like SNAT, but for a source IP that isn't fixed (e.g. a dynamic public IP)|
|`REDIRECT`|send the packet to a different local port/IP|
|`MARK`|tag the packet with a Netfilter mark for later routing decisions|

**matches** narrow down which packets a rule applies to:

|match|matches on|
|---|---|
|`-p` / `--protocol`|protocol (tcp, udp, icmp)|
|`--dport` / `--sport`|destination / source port|
|`-s` / `--source`|source IP|
|`-d` / `--destination`|destination IP|
|`-m state`|connection state (NEW, ESTABLISHED, RELATED)|
|`-m multiport`|multiple ports/ranges at once|
|`-m string`|a specific byte string in the payload|
|`-m limit`|rate limiting|
|`-m conntrack`|connection tracking info|
|`-m mac`|source MAC address|
|`-m iprange`|an IP range|

**example rules**, building a rule from criteria + target:

```bash
# allow inbound SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# allow inbound HTTP, explicit -m tcp match variant
sudo iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
```

`-A INPUT` appends the rule to the INPUT chain, `-p tcp --dport 22` is the match criteria (TCP, destination port 22), `-j ACCEPT` is the target (jump to ACCEPT, i.e. allow it).

#### the kernel's actual decision algorithm

the table above tells you _what exists_, this is _the order the kernel actually runs it in_, spelled out as pseudocode:

```
on packet arrival at a NIC:
    hook = PREROUTING
    run_table_chain(raw, PREROUTING, packet)
    run_table_chain(mangle, PREROUTING, packet)
    run_table_chain(nat, PREROUTING, packet)   # DNAT may rewrite dst here

    # --- routing decision, based on packet's dst AFTER any DNAT rewrite ---
    if packet.dst_ip == this_machine's_ip:
        path = LOCAL_DELIVERY
    else:
        path = FORWARD

    if path == LOCAL_DELIVERY:
        run_table_chain(mangle, INPUT, packet)
        run_table_chain(filter, INPUT, packet)
        if not dropped:
            deliver_to_local_process(packet)

    elif path == FORWARD:
        run_table_chain(mangle, FORWARD, packet)
        run_table_chain(filter, FORWARD, packet)
        if not dropped:
            run_table_chain(mangle, POSTROUTING, packet)
            run_table_chain(nat, POSTROUTING, packet)   # SNAT/MASQUERADE
            send_out_nic(packet)


on packet generated locally (by a process on this machine):
    hook = OUTPUT
    run_table_chain(raw, OUTPUT, packet)
    run_table_chain(mangle, OUTPUT, packet)
    run_table_chain(nat, OUTPUT, packet)     # rare, e.g. local DNAT redirects
    run_table_chain(filter, OUTPUT, packet)
    if not dropped:
        run_table_chain(mangle, POSTROUTING, packet)
        run_table_chain(nat, POSTROUTING, packet)
        send_out_nic(packet)


def run_table_chain(table, chain, packet):
    if table has no rules registered at this chain:
        return   # skip, nothing to check
    for rule in table[chain].rules:   # top to bottom
        if packet matches rule:
            apply target (ACCEPT / DROP / REJECT / DNAT / SNAT / LOG / ...)
            if target is terminating (not LOG):
                stop evaluating this chain
            break
    if no rule matched:
        apply table[chain].default_policy
```

the whole system reduces to: **one fork (did the packet arrive, or did this machine generate it) → one branch (is it for me, or does it need forwarding) → a sequential run through table/chain lookups, any one of which can kill the packet.**

#### four packets traced through it

**packet 1: incoming SSH**

|step|condition checked|result|
|---|---|---|
|entry|arrival|PREROUTING: raw, mangle, nat all run, no DNAT rule matches|
|decision|`dst_ip == this_machine?`|yes → `path = LOCAL_DELIVERY`|
|INPUT|mangle runs, filter runs|filter rule matches (`--dport 22`) → ACCEPT|
|end|not dropped|delivered to sshd|

**packet 2: local ping to google.com**

|step|condition checked|result|
|---|---|---|
|entry|generated locally|the arrival fork doesn't apply, this is the OUTPUT path from the start|
|OUTPUT|raw, mangle, nat, filter all run|no rules block it → ACCEPT (policy default)|
|decision|not dropped|proceed to POSTROUTING|
|POSTROUTING|mangle, nat run|no SNAT rule needed (already a real public IP)|
|end||sent out eth0|

**packet 3: LAN client → internet (masquerade)**

|step|condition checked|result|
|---|---|---|
|entry|arrival on eth1 (LAN)|PREROUTING: raw, mangle, nat run, no DNAT rule matches, dst stays 8.8.8.8|
|decision|`dst_ip == this_machine?`|no (8.8.8.8 ≠ router IP) → `path = FORWARD`|
|FORWARD|mangle, filter run|filter rule allows LAN→internet → ACCEPT|
|not dropped|proceed to POSTROUTING||
|POSTROUTING|mangle, nat run|MASQUERADE rule matches → src rewritten 192.168.1.50 → public IP|
|end||sent out eth0 (WAN)|

**packet 4: incoming port forward (DNAT)**

|step|condition checked|result|
|---|---|---|
|entry|arrival on eth0 (WAN), dst = public_ip:8080|PREROUTING: raw, mangle run, then nat runs and a DNAT rule matches → dst rewritten to 192.168.1.10:80|
|decision|`dst_ip == this_machine?`|checked **after** rewrite: 192.168.1.10 ≠ router IP → `path = FORWARD`|
|FORWARD|mangle, filter run|filter checks the rewritten dst 192.168.1.10:80, rule allows it → ACCEPT|
|not dropped|proceed to POSTROUTING||
|POSTROUTING|mangle, nat run|SNAT rule (if present) rewrites src so the reply routes back through the router|
|end||sent toward LAN, delivered to 192.168.1.10|

the one line that explains basically every "gotcha" in this whole topic: `if packet.dst_ip == this_machine's_ip` runs **after** the nat table already had a chance to rewrite it. that single ordering fact is why DNAT has to sit in PREROUTING, and why filter rules downstream always see post NAT addresses, never the original ones.

**feynman checks:**

1. in the algorithm, if a DNAT rule in PREROUTING rewrote a packet's dst to this machine's own IP (a local redirect, e.g. transparent proxy), which path would it take? → LOCAL_DELIVERY, because the routing decision check happens after the rewrite, so it'd go to INPUT instead of FORWARD, even though the packet originally arrived addressed elsewhere.
2. why does `run_table_chain` check "if table has no rules registered at this chain: return" instead of just always looping over an empty list? → functionally identical, an empty list loop would just immediately fall through with nothing to do, that line is just making explicit what the table matrix above already implies: some tables genuinely don't exist at some chains (e.g. `nat` has no INPUT chain).

---

### 1.28 System Logs

logs are the record of what already happened on a system, split across two main mechanisms depending on the distro/init system:

|location|covers|
|---|---|
|`/var/log/syslog` (or `/var/log/messages`)|general system activity, traditional plain text log|
|`/var/log/auth.log`|authentication events: logins, sudo usage, SSH sessions|
|`/var/log/kern.log`|kernel messages|
|`/var/log/dpkg.log` / `/var/log/apt/history.log`|package install/remove history|
|`journalctl`|systemd's own binary log store, covers services managed by systemd|

**journalctl usage:**

|command|does|
|---|---|
|`journalctl`|show the full journal, oldest first|
|`journalctl -u <service>`|logs for one specific systemd unit|
|`journalctl -f`|follow mode, like `tail -f`|
|`journalctl -b`|logs since the current boot only|
|`journalctl --since "1 hour ago"`|time filtered|

_why two separate systems coexist_: `journalctl`'s binary format supports structured querying (by unit, by boot, by time range) that plain text files can't do efficiently, but plenty of software still writes directly to flat text logs under `/var/log/` regardless of whether systemd is managing it, so a full picture of "what happened" usually means checking both.

Q: a service was definitely running an hour ago and isn't now, but `/var/log/syslog` shows nothing useful, what do you check next? A: `journalctl -u <service> --since "1 hour ago"`, since systemd managed services often log the actually useful detail (why it stopped, exit codes, restart attempts) into the journal rather than into `/var/log/syslog` at all.

---

### 1.29 Solaris

**Solaris** is a Unix operating system (not linux, a separate but related Unix lineage), originally developed by Sun Microsystems, now maintained by Oracle as **Oracle Solaris**. it matters here mainly as a point of comparison: same Unix philosophy and many overlapping concepts, but different tooling underneath.

**terminology/tooling differences worth knowing:**

|concept|linux|solaris|
|---|---|---|
|package management|`apt`/`dpkg` (Debian family) or `yum`/`rpm` (RHEL family)|`pkg` (image packaging system, IPS)|
|service management|systemd (`systemctl`)|SMF (Service Management Facility, `svcadm`/`svcs`)|
|kernel/version info|`uname -a`|`uname -a` works similarly, but Solaris also has `showrev -a` for a fuller hardware/OS revision report|
|filesystem|ext4 commonly|ZFS is native and deeply integrated (snapshots, pooled storage)|

_why this shows up in a linux focused module at all_: in real environments you don't get to assume every Unix like box you touch is linux, Solaris (and its descendants like illumos/OmniOS) still runs in plenty of legacy enterprise infrastructure, recognizing "this isn't quite linux" from the tooling available is itself a useful signal during enumeration.

Q: you SSH into a box, run `apt update` and it fails with "command not found," `systemctl` also doesn't exist, what's a reasonable next guess for what you're on? A: check `uname -a` and try `pkg` / `svcs`, if those resolve you're likely on Solaris (or a close relative) rather than a linux distro, different package manager and init system entirely.

---

### 1.30 Shortcuts

keyboard shortcuts that speed up regular shell use:

|shortcut|does|
|---|---|
|`[Ctrl] + C`|interrupt (kill) the currently running foreground command|
|`[Ctrl] + Z`|suspend the current process, send it to the background (resume with `fg`)|
|`[Ctrl] + D`|send EOF, exits the current shell/prompt|
|`[Ctrl] + L`|clear the terminal (same as running `clear`)|
|`[Ctrl] + R`|search command history interactively|
|`[Ctrl] + A`|jump cursor to the start of the line|
|`[Ctrl] + E`|jump cursor to the end of the line|
|`[Ctrl] + U`|delete from cursor to the start of the line|
|`[Ctrl] + K`|delete from cursor to the end of the line|
|`[Ctrl] + W`|delete the word before the cursor|
|`[Tab]` (x2)|autocomplete / list all matches|
|`[↑]` / `[↓]`|step through command history|

Q: you're mid way through typing a long command and realize the first half is wrong, fastest fix? A: `[Ctrl] + A` to jump to the start of the line, then `[Ctrl] + K` to wipe everything after the cursor to the end, faster than holding backspace or arrowing character by character.

---

## 2. Structured Breakdown (Cheat Sheet)

|#|section|core commands/files|one line takeaway|
|---|---|---|---|
|1.1|Linux Structure|`/etc`, `/var`, `/bin`, `/boot`|everything is a file, config lives in plain text|
|1.3|Introduction to Shell|bash|the shell is the CLI to the kernel, terminal is just the window it runs in|
|1.4|Prompt Description|`PS1`, `\u \h \w \@`|the prompt is a customizable template, not a fixed string|
|1.5|Getting Help|`man`, `--help`, `apropos`|`apropos <keyword>` finds tools you didn't know the name of|
|1.7|Navigation|`pwd`, `ls -la`, `cd -`|`cd -` jumps back to the previous directory, one entry history|
|1.8|Working with Files/Dirs|`touch`, `mkdir -p`, `mv`, `cp`|`mv` = rename, a rename is just a same directory move|
|1.9|Editing Files|`nano`, `vim`, `vimtutor`|vim is modal: keystrokes are commands, not text, until you enter Insert mode|
|1.10|Find Files and Dirs|`which`, `locate`, `find`|`locate` reads a cached index, `find` walks the live filesystem|
|1.11|File Descriptors|`>` `>>` `2>` `\|`|FD 0/1/2 = STDIN/STDOUT/STDERR, `2>/dev/null` silences errors only|
|1.13|Regular Expressions|`grep -E`, `^ $ .* \|`|`.*` between two terms is an ordered AND, not a true unordered AND|
|1.14|Permission Management|`chmod`, `chown`, SUID/SGID|octal = r(4)+w(2)+x(1) per actor, SUID runs as the file's owner|
|1.15|User Management|`useradd -m -s -G -g`, `/etc/shadow`|`useradd` needs `-m` explicitly, home dirs aren't automatic|
|1.16|Package Management|`apt update/upgrade/install`, `dpkg -l`|`update` refreshes the index, `upgrade` actually installs newer versions|
|1.17|Service/Process Mgmt|`systemctl`, `ps aux`, `kill`|`ps aux` shows every process system wide, bare `ps` only your session|
|1.18|Task Scheduling|`crontab -e`, `* * * * *`|5 fields: minute hour day month weekday, `*` = every value|
|1.19|Network Services|`/etc/exports`, SSH, NFS|`no_root_squash` on an NFS share = remote root stays root on the share|
|1.20|Working with Web Services|`python3 -m http.server`, `php -S`|php's server executes `.php` files, python's only serves static ones|
|1.21|Backup and Restore|`rsync -avz -e ssh`, cron|`-a` preserves perms/ownership/timestamps, plain `-r` doesn't|
|1.22|File System Management|`lsblk`, `fdisk -l`, `df -h`, `du -sh`|a filesystem isn't browsable until it's mounted onto a directory|
|1.23|Containerization|`docker run/ps/build/exec`|containers share the host kernel (namespaces+cgroups), no second kernel boots|
|1.24|Network Configuration|`ip a`, `ip route`, `/etc/hosts`|`/etc/hosts` is checked before DNS, can silently override resolution|
|1.25|Remote Desktop Protocols|VNC (5900), RDP/xrdp (3389)|VNC is unencrypted by default, normally tunneled over SSH|
|1.26|Linux Security|SELinux, AppArmor, fail2ban|MAC (SELinux/AppArmor) restricts even beyond what the file owner permits|
|1.27|Firewall Setup|`iptables -A INPUT -p tcp --dport 22 -j ACCEPT`|the "is this for me" check runs **after** NAT has already rewritten the destination|
|1.28|System Logs|`journalctl -u -f -b`, `/var/log/`|systemd managed services often log the useful detail only into the journal|
|1.29|Solaris|`pkg`, `svcs`, `showrev -a`|different Unix, different tooling: no `apt`, no `systemctl`|
|1.30|Shortcuts|`[Ctrl+C/Z/D/L/R/A/E/U/K/W]`|`[Ctrl+A]` then `[Ctrl+K]` is the fast way to wipe and retype a line|

---

## 3. Feynman Checks

1. **why does `2>/dev/null` silence errors but not regular output, when both start with a redirection arrow?** because the number before `>` specifies which file descriptor is being redirected, `2` is STDERR specifically, a bare `>` defaults to FD 1 (STDOUT), they're two independent streams that just happen to print to the same terminal by default, redirecting one doesn't touch the other.
    
2. **why is `chmod 754` unambiguous, i.e. why can't two different permission sets ever produce the same octal digit?** because each of r/w/x maps to a distinct power of 2 (4, 2, 1) within a 3 bit field per actor, and every combination of "which bits are set" sums to a unique value 0 through 7, there's no overlap possible, which is exactly why octal can represent the full 9 bit `rwxrwxrwx` string in just 3 digits with zero information loss.
    
3. **why does a DNAT rule have to live in `PREROUTING` specifically, and not, say, `INPUT`?** because the kernel's "is this packet for me" routing decision runs immediately after PREROUTING and checks the packet's _current_ destination IP, if DNAT hasn't rewritten it yet by that point, the decision gets made on the original, unrewritten address, which would break the entire mechanism, timing here isn't a convention, it's a hard ordering requirement.
    
4. **why does a container escape count as a fundamentally different threat class than a VM escape?** a container is just a normal process, isolated by kernel level namespaces/cgroups running on the _same_ kernel as the host, a VM escape has to defeat a hardware enforced hypervisor boundary, a container escape only has to defeat software policy within a shared kernel, which is a structurally thinner wall.
    
5. **why does `find` reliably see a file created one second ago while `locate` might not?** `find` walks the actual live filesystem on every single invocation, `locate` answers instantly because it's querying a prebuilt index (`updatedb`) that only refreshes on its own schedule, speed and freshness are a direct tradeoff between the two tools, not a bug in either one.