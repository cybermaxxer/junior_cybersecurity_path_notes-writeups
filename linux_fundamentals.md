
notes built from the htb academy linux module theory pages plus my own exercise log, so i keep the reasoning behind each command, not just the syntax.

## table of contents

1. [the 80/20 digest](#1-the-8020-digest)
   * [section 4: the prompt (ps1)](#section-4-the-prompt-ps1)
   * [section 6: environment basics](#section-6-environment-basics)
   * [section 7: file listing (ls -alps)](#section-7-file-listing-ls--alps)
   * [section 8: file listing sorted by time](#section-8-file-listing-sorted-by-time)
   * [section 9: vim / vimtutor recap](#section-9-vim--vimtutor-recap)
   * [section 10: find with filters + exec](#section-10-find-with-filters--exec)
   * [section 11: locate vs find, dpkg count](#section-11-locate-vs-find-dpkg-count)
   * [section 12: listening sockets + link scraping](#section-12-listening-sockets--link-scraping)
   * [section 13: grep filtering an ssh config](#section-13-grep-filtering-an-ssh-config)
   * [section 15: user management](#section-15-user-management)
   * [section 17: systemctl and units](#section-17-systemctl-and-units)
   * [section 18: dconf and d-bus activation](#section-18-dconf-and-d-bus-activation)
   * [section 19: nfs](#section-19-nfs)
   * [section 20: quick dev servers (npm/php)](#section-20-quick-dev-servers-npmphp)
   * [section 21: ssh key auth + rsync + cron backup lab](#section-21-ssh-key-auth--rsync--cron-backup-lab)
   * [section 22: disks](#section-22-disks)
2. [structured breakdown (cheat sheet)](#2-structured-breakdown-cheat-sheet)
3. [feynman checks](#3-feynman-checks)
---

## 1. the 80/20 digest

### section 4: the prompt (ps1)

`PS1` is just a variable holding a template string. bash reprints it every time it's ready for input, substituting the special backslash sequences for live values (username, path, colors...). nothing magic: it's string interpolation running on every prompt draw.

your edit:

```bash
export PS1="\[\e]0;\u@\h:\w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\@\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$"
```

|piece|meaning|
|---|---|
|`\[ ... \]`|tells bash "what's inside doesn't take screen space." without it, bash miscounts the cursor position once colors are involved and line wrapping breaks|
|`\e]0;...\a`|an OSC escape sequence that sets the terminal window/tab title. `\a` (bell char) closes it, it's not decorative, it's the literal terminator this escape sequence expects|
|`\u`|username|
|`\h`|hostname, short form (up to the first dot)|
|`\w`|current working directory, full path, `~` substituted for home|
|`${debian_chroot:+($debian_chroot)}`|parameter expansion, not a ps1 escape: "if `$debian_chroot` is set, output `(value)`, else output nothing." shows up when you're inside a chroot jail|
|`\033[01;32m`|ansi escape, raw terminal control code. `01` = bold, `32` = green foreground|
|`\033[00m`|reset all styling|
|`\@`|this is the one you added. it's a real ps1 escape for current time, 12h am/pm format. so you did add a clock, just not where you might expect visually (right after the hostname, no separating space)|
|`\$`|renders as `$` for a normal user, `#` for root, since it reads your effective uid|

Q: if you deleted the `\[ \]` wrappers around the color codes but kept everything else, would the prompt still look the same? A: it'd display fine but line wrapping/history recall would glitch, because bash would count the invisible color codes as visible characters.

### section 6: environment basics

`env` dumps your current shell's environment variables, the key/value pairs every child process inherits.

|command|what it tells you|
|---|---|
|`uname -m`|machine hardware architecture (e.g. `x86_64`)|
|`uname -s`|kernel name (e.g. `Linux`)|
|`env` \| grep `$MAIL`|path to your mail spool, historically `/var/mail/<user>`|
|`env` \| grep `$SHELL`|path to your default login shell binary|
|`ifconfig \| grep 1500`|filters for the mtu line; 1500 bytes is the standard ethernet frame ceiling|

Q: why does `uname -m` differ from `uname -s`? A: one reports the cpu architecture the kernel was built for, the other reports which kernel it is, two independent facts about the same box.

### section 7: file listing (`ls -alps`)

```bash
ls -alps
```

|flag|stands for|why it's here|
|---|---|---|
|`-a`|all|include dotfiles (hidden entries starting with `.`)|
|`-l`|long|full metadata: permissions, owner, group, size, mtime|
|`-p`|(no acronym, just "append indicator")|slaps a `/` on directory names so you can tell dirs from files at a glance without color|
|`-s`|size|prefixes each entry with the disk blocks it occupies, not the byte size shown by `-l`|

`ls -i | grep sudoers` adds inode numbers (`-i`), then filters for the sudoers file's inode. useful because filenames are just labels pointing at an inode, the inode is the actual file record on disk.

Q: could two different filenames share the same inode number? A: yes, that's exactly what a hard link is.

### section 8: file listing sorted by time

```bash
ls -halpsit | head -5
ls -halpsit | grep gshadow
```

adds two flags on top of section 7's set: `-h` (human readable sizes, so `4.0K` instead of `4096`) and `-t` (sort by modification time, newest first). piping to `head -5` just caps output at the first 5 lines of that sorted list.

Q: without `-t`, what order does `ls` default to? A: alphabetical by filename.

### section 9: vim / vimtutor recap

vimtutor is a guided, interactive copy of vim's built in tutorial, you run it and it walks you through a scratch file. core mental model: vim is modal, the same key does different things depending on which mode you're in.

|mode|how you enter it|what keys do|
|---|---|---|
|normal (default)|`Esc`|keys are commands (movement, delete, etc), not text|
|insert|`i`, `a`, `o`|keys type literal text|
|command line|`:` from normal|typed commands like save/quit/search|
|visual|`v` from normal|select text to operate on|

|key|action|
|---|---|
|`h j k l`|left, down, up, right (no arrow keys needed)|
|`x`|delete char under cursor|
|`dd`|delete current line|
|`u`|undo|
|`:w`|write (save)|
|`:q`|quit|
|`:wq`|save and quit|
|`/pattern`|search forward|

mnemonic for the 4 basic modes: **NICV**, normal, insert, command, visual.

Q: if you're stuck typing gibberish into your file mid vim session, what almost certainly happened? A: you're in insert mode and tried to run a normal mode command; hit `Esc` first.

### section 10: find with filters + exec

```bash
find / -name '*.conf' -size +25k ! -size +28k -newermt 2020-03-03 2>/dev/null -exec ls -halFsit {} \;
```

|piece|meaning|
|---|---|
|`-name '*.conf'`|filename glob match|
|`-size +25k`|strictly larger than 25kb|
|`! -size +28k`|negation: NOT larger than 28kb. combined with the previous test, this pins the range to (25k, 28k]|
|`-newermt 2020-03-03`|modified more recently than that date/time|
|`2>/dev/null`|redirects stderr (file descriptor 2) to the null device, so "permission denied" spam from folders you can't read doesn't clutter output. stdout (fd 1, the actual results) is untouched|
|`-exec CMD {} \;`|runs `CMD` once per matched file. `{}` is the placeholder find substitutes with the current match, `\;` is the escaped semicolon that terminates the `-exec` clause (unescaped, the shell would eat the `;` itself)|

`find / -name '*.bak' 2>/dev/null | nl` : `nl` just numbers each line of output, unrelated to find itself.

`which xxd` : prints the path of the binary that would run if you typed `xxd` (a hex dump tool), resolved by walking `$PATH`.

Q: why use `-exec ... \;` instead of `-exec ... +`? A: `\;` runs the command once per file (safer, slower), `+` batches as many matches as possible into fewer command invocations (faster, but behaves differently for commands sensitive to argument count).

### section 11: locate vs find, dpkg count

you tried `locate '.log' | wc -l` and got a wrong count. `locate` doesn't scan the filesystem live, it queries a prebuilt index (`mlocate.db`), refreshed by `updatedb`, usually via a daily cron job. if a file was created after the last `updatedb` run, `locate` simply doesn't know it exists yet. that's the exact mismatch you hit.

`find / -name '*.log' -type f 2>/dev/null | wc -l` walks the real filesystem live, so it's always current, just slower on a big disk. (note: `-type f` is the correct flag, meaning "regular file"; a stray `-type -f` in your log would actually throw an error since `-f` alone isn't a valid file type code.)

`dpkg -l | grep ii | wc -l` : `dpkg -l` lists package states as a two letter code. `ii` = desired state "install" + current state "installed", i.e. cleanly, fully installed packages. filtering `ii` excludes half installed or removed-but-config-remains packages.

Q: if you added a new `.log` file and immediately ran `locate '.log'`, would it show up? A: not necessarily, only after the next `updatedb` run, `find` would catch it immediately since it reads the disk directly.

### section 12: listening sockets + link scraping

```bash
ss -tnlp4 | grep -E '0.0.0.0:[0-9]+' | wc -l
```

|flag|meaning|
|---|---|
|`-t`|tcp sockets only|
|`-n`|numeric: show raw port numbers instead of resolving them to service names via `/etc/services`|
|`-l`|listening sockets only (not established connections)|
|`-p`|show the process/pid owning each socket|
|`4` (i.e. `-4`)|ipv4 only|

grepping `0.0.0.0:[0-9]+` finds services bound to all interfaces, as opposed to `127.0.0.1` (localhost only) or a specific interface ip.

you noted `ss -tlp | grep -i proftp` didn't work, but `ps aux | grep -i proftpd` did. makes sense: `ss` filters socket data (ports, addresses), it won't grep-match a process name unless that name literally appears in the socket line (it usually does via `-p`, but formatting/spacing can trip a naive grep). `ps aux` lists actual running processes by command name, a more direct match for "is proftpd running."

the curl pipeline:

```bash
curl -s https://www.inlanefreight.com | tr " " "\n" | tr "'" '"' | grep -E '"(https?://www.inlanefreight\.com)?/[^/][^"]*' | cut -d '"' -f 2 | sort -u | wc -l
```

|piece|job|
|---|---|
|`curl -s`|silent fetch, no progress meter cluttering output|
|`tr " " "\n"`|breaks the html into one "word" per line, crude but makes grep's job easier|
|`tr "'" '"'`|normalizes single quoted html attributes to double quotes, so one grep pattern catches both|
|`grep -E '"(https?://www.inlanefreight\.com)?/[^/][^"]*'`|matches quoted strings that look like a path or full url on that domain|
|`cut -d '"' -f 2`|pulls out just the url text between the quotes (field 2 when splitting on `"`)|
|`sort -u`|unique + sorted|
|`wc -l`|count|

this is link scraping without a real html parser, brittle but works for a quick recon count.

Q: why swap single quotes to double quotes before grepping? A: html can use either for attribute values, normalizing to one lets a single regex catch both cases.

### section 13: grep filtering an ssh config

looks like an `sshd_config` audit exercise:

|command|filters for|
|---|---|
|`grep -v #`|`-v` inverts the match, so this drops every commented line|
|`grep -E '^Permit'`|lines starting with "Permit" (e.g. `PermitRootLogin`)|
|`grep -E 'Authentication$'`|lines ending in "Authentication" (matches the directive name, not a value)|
|`grep 'Key'`|any line containing "Key" (e.g. `AuthorizedKeysFile`, `HostKey`)|
|`grep -E '^Password'*yes`|as written, the `*` sits outside the quotes so the shell glob-expands `yes` against filenames in your cwd instead of feeding it to grep as part of the pattern. the intent was almost certainly `grep -E '^Password.*yes'`, worth rerunning to make sure you got the real result and not a shell glob accident|
|`grep -E 'yes$'`|lines ending in "yes"|

Q: what's the practical difference between `grep -v #` and manually skimming the file for comments? A: none in output, but `-v` is instant and exact, no risk of eyeballing past a `#` buried mid line.

### section 15: user management

```bash
useradd -m -s /bin/bash -G sudo devs -g primarygroup123
```

|flag|meaning|
|---|---|
|`-m`|create the home directory (`/home/devs`), skipped by default otherwise|
|`-s /bin/bash`|login shell assigned to the account|
|`-G sudo`|supplementary group(s), comma separated if more than one. `sudo` group membership is what grants sudo rights on debian based systems|
|`-g primarygroup123`|primary group (must already exist)|
|`devs`|the username, the only positional (non flag) argument|

`usermod --lock` : locks the account by prefixing the password hash in `/etc/shadow` with `!`, the account can't authenticate via password until unlocked, but existing sessions/keys aren't killed.

`su root --command "cat /etc/shadow"` : switches to root and runs a single command non interactively, then returns. `/etc/shadow` holds password hashes, readable only by root, which is exactly why this needed the `su` first.

Q: does locking a user account with `usermod --lock` kill their currently open ssh session? A: no, it only blocks new password based logins, existing sessions and key based auth (unless also disabled) keep working.

### section 17: systemctl and units

`systemctl list-units` lists all currently loaded units (services, sockets, mounts, etc) and their state. inside the pager, `/pattern` opens a forward search, same muscle memory as vim/less, `n` for next match.

mental model: `systemctl` doesn't run programs directly, it talks to `systemd` (pid 1), which owns starting, stopping, and supervising every unit. `list-units` shows what's currently loaded into that supervisor, `list-unit-files` (a different subcommand) shows everything installed regardless of current load state.

Q: what's the difference between a unit being "loaded" and "active"? A: loaded means systemd parsed its definition into memory, active means it's actually running/succeeded, a unit can be loaded but inactive.

### section 18: dconf and d-bus activation

```bash
locate dconf.service
cat /usr/share/dbus-1/services/ca.desrt.dconf.service
```

dconf is gnome's low level configuration database (think: a structured settings store other apps read/write to). the key idea in that `.service` file: dconf isn't started at boot like a normal daemon, it's d-bus activated, meaning d-bus starts it on demand the first time some process actually sends it a request over the bus. saves resources by not running things nobody's using yet.

Q: if no gnome app ever queries dconf during a session, does dconf.service ever start? A: no, that's the whole point of d-bus activation, it stays dormant until requested.

### section 19: nfs

nfs (network file system) lets a remote directory be mounted and used as if it were local. config lives in `/etc/exports`, one line per shared directory plus its access rules.

|option|meaning|
|---|---|
|`rw`|clients get read + write access|
|`ro`|read only|
|`no_root_squash`|the client's root user keeps full root privileges on the share (dangerous if you don't fully trust the client)|
|`root_squash`|maps the client's root down to a normal unprivileged user on the share (the safer default)|
|`sync`|confirms a write only after it's actually committed to disk, safer, slower|
|`async`|acknowledges writes before they're confirmed on disk, faster, risk of corruption on a crash|

example flow: `sudo apt install nfs-kernel-server`, check it with `systemctl status nfs-kernel-server`, share a folder by appending a line like `/home/user/share client_host(rw,sync,no_root_squash)` to `/etc/exports`, then on the client side `mount server_ip:/remote/path ~/local_mount` to attach it.

Q: as an attacker, why would `no_root_squash` on a share you can mount be a big deal? A: you could create a setuid binary owned by root from your own machine, then execute it via the mount and get root on the nfs server.

### section 20: quick dev servers (npm/php)

two ways to spin up a throwaway http file server for a quick test or exfil/recon setup.

|approach|command|note|
|---|---|---|
|node/npm|`npm install http-server` then `npx http-server -p 8080`|`npm` is node's package manager, `npx` runs a package's binary without a permanent global install. `-p` sets the port|
|php|`php -S 127.0.0.1:8080`|`-S` starts php's built in single threaded dev server, no apache/nginx needed, binds to the given host:port|

Q: why bind php's dev server to `127.0.0.1` instead of `0.0.0.0`? A: `127.0.0.1` only accepts local connections, `0.0.0.0` would expose it to the whole network, wider attack surface for something meant to be a quick local test.

### section 21: ssh key auth + rsync + cron backup lab

end to end: passwordless ssh, then an automated rsync backup on a schedule.

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t rsa -b 2048 -f ~/key -N ""
cat ~/key.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
ssh -i ~/key -o StrictHostKeyChecking=no $USER@127.0.0.1 "echo 'Manual key installed'"
```

|piece|meaning|
|---|---|
|`chmod 700 ~/.ssh`|owner rwx, nobody else. ssh refuses to use a key setup if permissions are too loose, this isn't optional|
|`ssh-keygen -t rsa -b 2048 -f ~/key -N ""`|`-t` key type, `-b` key size in bits, `-f` output path, `-N ""` empty passphrase so the key can be used unattended (cron can't type a passphrase)|
|`>> authorized_keys`|append the public key so that key is now trusted for login|
|`chmod 600`|owner read/write only, ssh enforces this too|
|`-i ~/key`|which private key to authenticate with|
|`-o StrictHostKeyChecking=no`|skips the "are you sure you want to continue connecting" host key prompt. fine for a disposable lab, a real MITM risk on a production box since you're disabling the check that detects a swapped host key|

```bash
rsync -avz -e "ssh -i /home/$USER/key -o StrictHostKeyChecking=no" /home/$USER/to_back/ $USER@127.0.0.1:/home/$USER/to_sync/
```

|flag|meaning|
|---|---|
|`-a`|archive mode, bundles recursive copy + preserve permissions, timestamps, symlinks, ownership. the "just do the sane backup thing" flag|
|`-v`|verbose|
|`-z`|compress data during transfer|
|`-e "..."`|overrides the remote shell rsync uses, here a custom ssh command carrying the identity file|

cron:

```
* * * * * /home/$USER/backuper.sh
```

five fields = minute, hour, day of month, month, day of week. all `*` means "every minute, every hour, every day" i.e. runs once a minute.

Q: why does the ssh key need an empty passphrase (`-N ""`) specifically for this cron use case? A: cron runs non interactively, nothing is there to type a passphrase when prompted, so a passphrase protected key would just hang/fail the job.

### section 22: disks

`sudo fdisk -l` lists disks and their partition tables (requires root to read partition details on some setups). `lsblk` gives a tree view of block devices and how they nest (disk → partitions → mount points), no root needed, easier to scan at a glance.

Q: which command would you reach for to quickly see which partition is mounted where? A: `lsblk`, its tree output shows the mountpoint column directly.

---

## 2. structured breakdown (cheat sheet)

|command / flag|explanation|practical use|
|---|---|---|
|`\[ \]` in ps1|marks enclosed chars as zero width for bash's line length counter|keeps prompt line wrapping correct when colors are used|
|`\u \h \w \@ \$`|ps1 escapes: user, host, cwd, 12h time, prompt char|build an informative custom prompt|
|`ls -alps` / `-t` / `-i` / `-h`|all, long, slash dirs, block size, sort by time, inode, human sizes|inspect files/dirs with full metadata|
|`find ... -size +N -newermt DATE -exec ... \;`|filter by size/date, run a command per match|targeted recon across a filesystem|
|`2>/dev/null`|discard stderr only|hide permission denied noise, keep real results|
|`locate` vs `find`|indexed lookup (fast, can be stale) vs live scan (slow, always current)|pick based on whether freshness matters|
|`dpkg -l \| grep ii`|list fully installed packages|quick software inventory|
|`ss -tnlp4`|tcp, numeric, listening, show pid, ipv4|enumerate open ports and owning processes|
|`grep -v` / `-E` / `^` `$`|invert match, extended regex, anchor start/end|filter config files down to what matters|
|`useradd -m -s -G -g`|create user with home, shell, groups|provision an account with intended access|
|`usermod --lock`|disable password auth on an account|freeze an account without deleting it|
|`systemctl list-units`|show loaded units and state|check what's running under systemd|
|d-bus activation|service starts on first request, not at boot|saves resources for rarely used services (e.g. dconf)|
|`/etc/exports` (`rw/ro`, `root_squash`)|nfs share definitions and access rules|share/mount directories over the network|
|`npx http-server -p` / `php -S host:port`|one line disposable file servers|quick local http server for testing or transfer|
|`ssh-keygen -N ""` + `authorized_keys`|passwordless key based login|required for unattended/cron ssh usage|
|`rsync -avz -e "ssh ..."`|archive, verbose, compressed sync over a custom ssh command|scripted backups|
|cron `* * * * *`|minute hour day month weekday|schedule the backup script every minute|
|`fdisk -l` / `lsblk`|partition table dump / block device tree|inspect disk layout|

---

## 3. feynman checks

1. **your `ss -tlp | grep -i proftp` attempt failed but `ps aux | grep -i proftpd` worked. why does that make sense given what each tool actually reads?** `ss` reports socket/network state, it only shows a process name if `-p` successfully resolves it and the string format matches your grep exactly. `ps aux` reads the process table directly, listing the actual running command names, a much more direct target for a name based grep.
    
2. **you hit a wrong count with `locate '.log' | wc -l` but got it right with `find`. what's the underlying reason, and when would you deliberately still prefer `locate`?** `locate` queries a periodically rebuilt index, not the live disk, so anything created/deleted since the last `updatedb` run won't be reflected. you'd still prefer `locate` on a huge filesystem where speed matters more than perfect freshness, e.g. searching for old, unchanging files.
    
3. **in the section 21 lab, why does the ssh key need `-N ""` and why does `~/.ssh` need `chmod 700` specifically, not just "restrictive enough"?** `-N ""` sets an empty passphrase because cron runs the backup script unattended, nothing can type a passphrase at 3am. `chmod 700` isn't arbitrary either: ssh actively checks the permission bits on `.ssh` and its key files, and refuses to use them if group/other have any access, it's a hardcoded safety check, not a suggestion.
