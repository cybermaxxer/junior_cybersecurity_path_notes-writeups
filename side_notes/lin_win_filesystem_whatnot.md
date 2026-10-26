# linux vs windows filesystems: sysadmin and cybersec conspectus

*my study notes for sysadmin and cybersec work. sources checked 2026-10-25. scope: debian/ubuntu, windows 10/11 client.*

## contents

- [1. the 80/20 digest](#1-the-8020-digest)
- [2. linux filesystem: where things live and why](#2-linux-filesystem-where-things-live-and-why)
- [3. windows filesystem: where things live and why](#3-windows-filesystem-where-things-live-and-why)
- [4. the windows registry](#4-the-windows-registry)
- [5. where software lives: apt, snap, flatpak and windows](#5-where-software-lives-apt-snap-flatpak-and-windows)
- [6. file manager commands: bash and powershell](#6-file-manager-commands-bash-and-powershell)
- [7. structured breakdown (cheat sheet)](#7-structured-breakdown-cheat-sheet)
- [8. feynman checks](#8-feynman-checks)
- [9. omitted chuff](#9-omitted-chuff)
- [10. watch it explained](#10-watch-it-explained)

## 1. the 80/20 digest

> **tl;dr:** linux has one tree rooted at `/`, config lives in plain text under `/etc`, and almost everything (disks, devices, processes) shows up as a file. windows has one tree per volume (`C:\`), config lives in a binary database (the registry) plus `ProgramData` and `AppData`, and access is controlled by NTFS ACLs. on any unknown machine i ask three things: where is the config, where are the logs, where does it autostart? attackers and defenders both care about those same three places.

### the mental model i'm building

the two systems look different because they were solving different problems.

- **linux: "everything is a file".** unix wanted one small set of tools (`cat`, `grep`, `chmod`) to work on everything. for that, every thing needs a path, so there is one tree. a new disk doesn't get its own letter, it gets *mounted* (grafted) onto a folder in the tree.
- **windows: drive letters are a legacy skin.** dos copied cp/m and gave each device a letter (A: and B: were floppies, so C: became the first hard disk). the NT kernel underneath uses an object namespace (`\Device\HarddiskVolume3`), and `C:` is just a shortcut into it. the letters stay because every app ever written expects them.
- **config: text vs database.** unix configs are plain text so i can edit, diff and grep them. windows 3.x scattered `.ini` files everywhere, so windows 95/NT centralized settings into the **registry**, a hierarchical database. the price: it's binary, harder to diff, and one corrupt hive hurts.
- **permissions: three bits vs lists.** unix = owner/group/other with read, write, execute. windows = a list of entries (ACEs) per file, each tied to a user or group SID, with inheritance from the parent folder.

*Q: Why one root on linux? A: every device needs a path to be opened.*

### the map: side by side

| what i'm looking for | linux (debian/ubuntu) | windows 10/11 | why they differ |
| :-- | :-- | :-- | :-- |
| root | `/` | `C:\` (one per volume) | unix mounts into one tree; dos gave devices letters |
| path separator | `/` | `\` (most apis also accept `/`) | dos already used `/` for command switches |
| case | case sensitive (`A.txt` is not `a.txt`) | case insensitive but preserving | unix treats names as raw bytes; windows is friendly to humans |
| system binaries | `/usr/bin`, `/usr/sbin` (`/bin` and `/sbin` link into them) | `C:\Windows\System32` | linux splits by role; windows keeps the core OS in one folder |
| system config | `/etc` (text) | registry + `C:\ProgramData` | text for editing vs database for central control |
| user home | `/home/<name>` | `C:\Users\<name>` | same idea, different name |
| user config | dotfiles (`~/.bashrc`, `~/.config/`) | `AppData\Roaming`, `AppData\Local`, `NTUSER.DAT` | linux hides by name; windows hides by attribute |
| logs | `/var/log` plus the systemd journal | `C:\Windows\System32\winevt\Logs\*.evtx` | text vs structured binary events |
| temp | `/tmp`, `/var/tmp` | `%TEMP%`, `C:\Windows\Temp` | per user temp vs system temp on windows |
| devices | `/dev` | `\Device\...` (object manager, not browsable in explorer) | linux exposes devices as files |
| live kernel view | `/proc`, `/sys` | none; use `Get-Process`, `Get-CimInstance` | windows exposes this through apis, not folders |
| installed apps | `/usr`, `/opt`, `/var/lib/dpkg` | `C:\Program Files`, `C:\Program Files (x86)`, registry `Uninstall` keys | linux spreads files by type; windows keeps one folder per app |
| hidden files | name starts with `.` | the Hidden and System attributes | a dot is a convention; an attribute is a flag in NTFS |
| who may run it | execute bit (`x`) | ACL plus file type (`.exe`, `.ps1`) | unix asks the file; windows asks the extension |

*Q: Why are dotfiles hidden? A: `ls` skips names starting with a dot by convention.*

### the three questions i ask on any machine

| question | linux answer | windows answer |
| :-- | :-- | :-- |
| where is the config? | `/etc`, `~/.config` | registry (`HKLM\SOFTWARE`, `HKCU\Software`), `ProgramData`, `AppData` |
| where are the logs? | `/var/log`, `journalctl` | Event Viewer, `winevt\Logs` |
| where does it autostart? | systemd units, cron, `~/.bashrc`, `~/.config/autostart` | `Run` keys, Startup folders, services, scheduled tasks |

*Q: What do all three questions share? A: They reveal what the system does without me.*

### my first two commands, broken down

| command | piece | what it does and why |
| :-- | :-- | :-- |
| `ls -la /` | `ls` | "list" (unix names are short on purpose: typing was slow and painful in the 1970s) |
|  | `-l` | long format: permissions, owner, size, date |
|  | `-a` | "all": includes dotfiles, which `ls` hides by default |
|  | `/` | the path to list: the root of the tree |
| `Get-ChildItem -Force C:\` | `Get` | the verb. powershell has an approved verb list (`Get`, `Set`, `New`, `Remove`), so commands are guessable |
|  | `ChildItem` | the noun: "the items inside a container", which covers folders, registry keys and more |
|  | `-Force` | show hidden and system items (a plain `Get-ChildItem` skips them) |
|  | `C:\` | the path to list: the root of the volume |

the verb plus noun shape in powershell is a real pattern, not a coincidence: i can guess `Remove-Item` or `New-Item` without looking. the short names (`ls`, `dir`, `gci`) are aliases, so those are memorization, not logic.

*Q: Why `-Force` in powershell? A: Hidden and system items are skipped otherwise.*

### security angle: the same places, two sides

- **how it gets exploited:** autostart locations and writable folders. malware drops a `Run` key or a cron entry so it survives a reboot. a world writable folder in the `PATH` lets any user plant a fake `ls` or `cmd.exe` that an admin later runs.
- **how i defend it:** least privilege on those folders, and auditing. i watch the autostart spots for changes (auditd or `inotifywait` on linux, registry auditing and Sysmon on windows) because a new entry there is rarely an accident.

*Q: Why do attackers love autostart spots? A: Persistence: they survive reboots.*

## 2. linux filesystem: where things live and why

the linux tree isn't random: the Filesystem Hierarchy Standard (FHS) sorts every folder by two questions. does it change while the system runs? and is it specific to this machine? once i can answer those two, i can guess where almost anything lives.

### the rule that explains the whole tree

|  | shareable between machines | this machine only |
| :-- | :-- | :-- |
| **static** (changes when an admin changes it) | `/usr`, `/opt` | `/etc`, `/boot` |
| **variable** (changes while running) | `/var/mail`, `/var/spool` | `/var/log`, `/var/lib`, `/run`, `/tmp` |

this is practical, not academic. `/usr` is static, so i can mount it read only. `/var` is variable, so i put it on its own partition: a runaway log can fill `/var` without freezing the whole system. for backups i care about `/etc`, `/home` and `/var/lib`, not `/usr`, because i can reinstall `/usr` from packages.

*Q: Why give `/var` its own partition? A: A full log disk can't break the root.*

### the tree, one line each

```text
/
├── bin   -> usr/bin     symlink on modern debian/ubuntu (usrmerge)
├── sbin  -> usr/sbin    symlink
├── lib   -> usr/lib     symlink
├── boot                 kernel (vmlinuz), initramfs, grub
├── dev                  device files (disks, tty, null, random)
├── etc                  system wide config, mostly plain text
├── home                 user home folders (/home/<name>)
├── root                 the root user's home
├── media, mnt           mount points (auto removable vs manual)
├── opt                  self contained third party apps
├── proc                 virtual: processes and kernel info
├── run                  virtual (tmpfs): runtime state since boot
├── srv                  data this machine serves (web, ftp)
├── sys                  virtual: kernel objects and hardware (sysfs)
├── tmp                  world writable scratch space
├── usr                  the bulk of installed programs (static)
├── var                  changing data: logs, caches, databases, spools
└── snap                 mount points for snap packages (ubuntu)
```

### why each top level folder exists

| folder | what lives there | the why |
| :-- | :-- | :-- |
| `/usr` | `bin`, `sbin`, `lib`, `share`, `local` | early unix ran out of space on the first disk, so programs moved to a second one mounted at `/usr`. the story says it was "user" at first and "unix system resources" later. today `/bin`, `/sbin`, `/lib` are symlinks into it, so one tree can be mounted read only |
| `/etc` | `passwd`, `fstab`, `ssh/`, `apt/`, `systemd/` | original meaning was "et cetera" (leftovers). i remember it as "editable text config", which is a memory aid, not the history |
| `/var` | `log`, `lib`, `cache`, `spool`, `www` | "variable": everything the system writes while running. `/var/lib` holds app state (databases, the dpkg package db) |
| `/home` and `/root` | user files and dotfiles | root's home is `/root`, not `/home/root`, because `/home` may be a separate disk that isn't mounted in rescue mode, and root must always be able to log in |
| `/boot` | `vmlinuz-*`, `initrd.img-*`, `grub/grub.cfg` | must be readable by the bootloader before the real root is mounted, so it is often its own small partition. on UEFI there is also `/boot/efi` |
| `/dev` | `sda`, `nvme0n1`, `null`, `urandom`, `tty` | "everything is a file" applied to hardware. writing to `/dev/null` discards data, reading `/dev/urandom` gives random bytes |
| `/proc` | `<pid>/`, `cpuinfo`, `meminfo` | not on disk. the kernel generates it on read, so `cat /proc/meminfo` is asking the kernel a question through a file |
| `/sys` | devices, drivers, power | like `/proc`, but a tidy view of kernel objects (sysfs) |
| `/run` | pid files, sockets, `systemd/` | a RAM filesystem (tmpfs) wiped on every boot, so stale pid files can't survive a crash. `/var/run` is a symlink to it |
| `/tmp` vs `/var/tmp` | scratch files | `/tmp` is cleared on reboot (and may live in RAM); `/var/tmp` survives reboots |
| `/opt` | `/opt/google/chrome` | a vendor ships a self contained folder instead of spreading files across `/usr` |
| `/usr/local` | `bin`, `lib`, `share` | software i installed by hand, outside the package manager, so `apt` never overwrites it |
| `/media` vs `/mnt` | USB stick vs manual mount | `/media` is for auto mounted removable media, `/mnt` is a free slot for me |
| `/lost+found` | recovered fragments | `fsck` drops orphaned file pieces here on ext filesystems. the `+` is just to be unlikely to collide with a real name |

*Q: Why is `/proc` always 0 bytes? A: It isn't stored; the kernel generates it.*

### the folders and files i check first (sysadmin and cybersec)

| path | what it holds | why i care |
| :-- | :-- | :-- |
| `/etc/passwd` | one line per user: `name:x:uid:gid:comment:home:shell` | world readable on purpose: tools like `ls -l` need it to turn a uid into a name. the `x` means the hash is in shadow |
| `/etc/shadow` | password hashes (root and group `shadow` only) | moved out of `passwd` so normal users can't read hashes and crack them offline |
| `/etc/group`, `/etc/sudoers`, `/etc/sudoers.d/` | groups and who may use sudo | a line added here is privilege escalation. always edit sudoers with `visudo`, which syntax checks before saving |
| `/etc/ssh/sshd_config` | ssh server settings | `PermitRootLogin`, `PasswordAuthentication` are the first things an auditor checks |
| `~/.ssh/authorized_keys` | public keys allowed to log in as that user | an attacker's key here is a quiet backdoor with no password |
| `/etc/fstab` | what gets mounted at boot, with options | where `noexec,nosuid,nodev` hardening goes |
| `/etc/hosts`, `/etc/resolv.conf`, `/etc/hostname` | local name resolution and hostname | a rewritten `hosts` line silently redirects a domain |
| `/etc/crontab`, `/etc/cron.d/`, `/var/spool/cron/crontabs/` | scheduled jobs (system and per user) | classic persistence: a job that re-creates the malware every minute |
| `/etc/systemd/system/` and `/usr/lib/systemd/system/` | admin units and overrides vs vendor units | a new `.service` here that starts on boot is modern persistence |
| `/etc/ld.so.preload` | libraries loaded into every program | rare and powerful: legit use is almost zero, so any content is suspicious |
| `/etc/profile.d/`, `~/.bashrc`, `~/.profile` | shell startup scripts | run on every login: another persistence spot |
| `/etc/pam.d/` | authentication stack | change here changes how logins work for everything |
| `/var/log/auth.log` | logins, sudo, ssh (needs rsyslog; otherwise use `journalctl`) | the first file for any intrusion question |
| `/var/log/syslog`, `/var/log/kern.log` | general and kernel messages | crashes, usb plug events, firewall drops |
| `/var/log/wtmp`, `/var/log/btmp` | successful and failed logins, binary | read with `last` and `lastb`, not `cat` |
| `/var/log/dpkg.log`, `/var/log/apt/history.log` | what got installed and when | answers "when did this package appear?" |
| `~/.bash_history` | commands typed by that user | timeline of what a user (or attacker) did. written when the shell exits, and easy to clear |
| `/proc/<pid>/cmdline`, `environ`, `exe`, `fd/` | a live process's arguments, environment, binary, open files | shows what a process really is, even if it renamed itself |
| `/var/lib/dpkg/status` | the package database | the truth about what is installed |
| `/boot/grub/grub.cfg` | boot menu and kernel parameters | someone with write access can boot into a root shell |

*Q: Why is `/etc/passwd` world readable? A: Tools map uids to names with it.*

### permissions: reading the string

every `ls -l` line starts with ten characters. the first is the type, then three groups of three: owner, group, other.

```text
-rw-r----- 1 root shadow 1342 Oct 20 09:12 /etc/shadow
drwxrwxrwt 18 root root  4096 Oct 25 08:01 /tmp
```

| piece | meaning |
| :-- | :-- |
| `-` or `d` first | `-` file, `d` directory, `l` symlink |
| `rw-` | owner may read and write (r=4, w=2, x=1, so this is 6) |
| `r--` | group may read (4) |
| `---` | everyone else gets nothing (0) |
| so `/etc/shadow` is 640 | `chmod 640` sets exactly this |
| `t` at the end of `/tmp` | the sticky bit: anyone can create files, but only the owner (or root) can delete or rename their own |
| `s` in place of `x` | setuid or setgid: the program runs with its owner's or group's rights, not mine |
| `x` on a directory | means "may enter" (traverse), not "may execute" |

why the sticky bit exists: without it, any user could delete another user's files in `/tmp`, because write on a directory means "may add and remove entries". that's the one permission rule that surprises everyone.

*Q: What does `x` mean on a folder? A: Permission to enter it.*

### commands for this section, broken down

| command | piece | what it does and why |
| :-- | :-- | :-- |
| `ls -ld /tmp /etc/shadow` | `-l` | long format |
|  | `-d` | "directory": show the folder itself, not what is inside it. without it `ls -l /tmp` lists the contents |
| `df -hT` | `df` | "disk free": usage per mounted filesystem |
|  | `-h` | human readable sizes (G, M) instead of 1K blocks |
|  | `-T` | adds the filesystem type: `ext4`, `tmpfs`, `proc`. this is how i see which folders are real disks and which are virtual |
| `findmnt /` | `findmnt` | shows the mount tree. with a path it shows what is mounted there. no flag logic to memorize, the name says it |
| `readlink /proc/$$/exe` | `$$` | a bash variable holding the pid of the current shell |
|  | `/proc/<pid>/exe` | a symlink to the binary that process runs |
|  | `readlink` | prints where a symlink points. result: `/usr/bin/bash` |

the flag letters here (`-l`, `-d`, `-h`, `-T`) mostly come from the first letter of the word they mean (long, directory, human, Type), but that's a habit, not a rule. when one confuses me, i run `man ls` and search with `/`.

*Q: Which flag adds the filesystem type to `df`? A: `-T`.*

### security angle: exploit and defend

- **how it gets exploited:** a world writable place where code can run. an attacker drops a payload in `/tmp` or `/dev/shm` and executes it. a writable script called by a root cron job is the same trick: edit the script, wait one minute, get root. reading `/proc/<pid>/environ` of a process i own can leak secrets passed as environment variables.
- **how i defend it:** mount `/tmp`, `/var/tmp` and `/dev/shm` with `nosuid,nodev,noexec` in `/etc/fstab` (`nosuid` ignores setuid bits, `nodev` ignores device files, `noexec` blocks running programs from there). downside: some installers need to execute from `/tmp`, so test first. add auditd rules on `/etc/passwd`, `/etc/shadow` and `/etc/sudoers`, and run a file integrity tool like AIDE so changes to `/etc` show up.

*Q: Why `noexec` on `/tmp`? A: Attackers drop payloads in world writable folders.*

## 3. windows filesystem: where things live and why

windows sorts folders by *who owns the thing*, not by how it changes. the OS lives in `C:\Windows`, apps in `C:\Program Files`, machine wide app data in `C:\ProgramData`, and each person's stuff in `C:\Users\<name>`. if i can answer "is this the OS, an app, the machine, or a user?", i know the folder. the default filesystem is NTFS, which is what gives windows ACLs, streams and journaling.

### the tree, one line each

```text
C:\
├── Windows                 the OS itself
│   ├── System32            64 bit core binaries, drivers, config, logs
│   ├── SysWOW64            32 bit core binaries (the name looks backwards)
│   ├── WinSxS              component store: every version of every system component
│   ├── Prefetch            traces of programs that ran (clients)
│   ├── Temp                system temp
│   └── SoftwareDistribution  windows update downloads
├── Program Files           64 bit apps
├── Program Files (x86)     32 bit apps
├── ProgramData             machine wide app data (hidden)
├── Users
│   ├── <name>              one profile per person
│   ├── Public              shared between users
│   └── Default             template copied for every new profile
├── pagefile.sys, hiberfil.sys, swapfile.sys   hidden system files
├── $Recycle.Bin, System Volume Information    hidden system folders
└── Windows.old             previous install, left after a major upgrade
```

### why each one exists

| folder | what lives there | the why |
| :-- | :-- | :-- |
| `C:\Windows\System32` | `.exe`, `.dll`, `drivers\`, `config\`, `winevt\Logs\`, `Tasks\` | it was the 32 bit system folder in the 90s. when 64 bit arrived, microsoft kept the name for the 64 bit files so old software still finds them |
| `C:\Windows\SysWOW64` | 32 bit versions of core files | WOW64 (windows on windows 64) feeds 32 bit programs this folder when they ask for `System32`. so the name is a compatibility trick, not a typo |
| `C:\Windows\WinSxS` | hard links and versions of system components | "side by side": fixes DLL hell by keeping many versions at once. it looks huge but is mostly hard links. never delete by hand, use `DISM /Online /Cleanup-Image /StartComponentCleanup` |
| `C:\Program Files` and `(x86)` | one folder per installed app | separating 32 and 64 bit keeps their registry views and DLLs apart. by default only admins and TrustedInstaller can write here |
| `C:\ProgramData` | settings, caches, logs shared by all users | replaced old `C:\Documents and Settings\All Users`. hidden by default. regular users can create things here, which matters for security |
| `C:\Users\Public` | shared files | the one profile folder meant for sharing |
| `C:\Users\Default` | template profile | copied when a user logs in for the first time, so changing it changes every future user |
| `C:\Windows.old` | previous windows | a rollback safety net, auto deleted after about 10 days. it also contains old hives and user data, so wipe it on decommission |
| `C:\$Recycle.Bin` | deleted items per user SID | deleted does not mean gone: files stay until emptied |
| `C:\System Volume Information` | restore points, volume shadow copies | readable only by SYSTEM. shadow copies are why ransomware tries to delete them |
| `pagefile.sys` | virtual memory on disk | can hold fragments of anything that was in RAM |
| `hiberfil.sys` | RAM image saved on hibernate and fast startup | a full memory snapshot at rest: forensic gold and a secrets leak if the disk isn't encrypted |

old paths still work because of **junctions** (NTFS shortcuts at the filesystem level). `C:\Documents and Settings` points to `C:\Users`, and `C:\Users\All Users` points to `C:\ProgramData`. that's how a 2003 installer survives on windows 11.

*Q: Why does `SysWOW64` hold 32 bit files? A: Compatibility: 32 bit apps get redirected.*

### inside a user profile

| path (under `C:\Users\<name>`) | what lives there | the why |
| :-- | :-- | :-- |
| `Desktop`, `Documents`, `Downloads` | the user's files | visible folders; `Downloads` is where untrusted files land first |
| `AppData\Roaming` (`%APPDATA%`) | small settings that follow the user | "roaming" because on a domain, a roaming profile copies it to every machine the user logs into |
| `AppData\Local` (`%LOCALAPPDATA%`) | big, machine specific data: caches, browser profiles, `Temp\` | stays on this machine because it's too large or hardware specific to sync |
| `AppData\LocalLow` | data for low integrity (sandboxed) processes | sandboxes run with a low integrity level and can't write to normal folders, so they get their own |
| `NTUSER.DAT` | the user's registry hive (HKCU) | loaded at logon, locked while the user is logged in |
| `AppData\Local\Microsoft\Windows\UsrClass.dat` | the HKCU\\Software\\Classes hive | file associations and shell data per user |
| `AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup` | per user autostart folder | anything dropped here runs at logon |
| `AppData\Roaming\Microsoft\Windows\Recent` | shortcuts to recently opened files | shows what the user touched, when |
| `AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt` | powershell command history | the windows twin of `.bash_history`, and it survives closing the window |
| `AppData\Roaming\Microsoft\Credentials`, `...\Protect` | DPAPI protected secrets and keys | encrypted with the user's logon secret, so attackers who get a session go after them |

the same locations as environment variables, so i can use them in scripts:

| variable | usual value | linux twin |
| :-- | :-- | :-- |
| `%SystemRoot%` or `%WINDIR%` | `C:\Windows` | `/` (kind of) |
| `%ProgramFiles%` | `C:\Program Files` | `/usr` |
| `%ProgramData%` | `C:\ProgramData` | `/var/lib`, `/etc` |
| `%USERPROFILE%` | `C:\Users\<name>` | `$HOME` |
| `%APPDATA%` | `...\AppData\Roaming` | `~/.config` |
| `%LOCALAPPDATA%` | `...\AppData\Local` | `~/.cache`, `~/.local/share` |
| `%TEMP%` | `...\AppData\Local\Temp` | `/tmp` |

in powershell the same variables live in the `env:` drive: `$env:APPDATA`, `$env:TEMP`.

*Q: Which AppData folder follows me between domain PCs? A: Roaming.*

### files and folders i check first (sysadmin and cybersec)

| path | what it holds | why i care |
| :-- | :-- | :-- |
| `C:\Windows\System32\config\` | registry hive files: `SAM`, `SECURITY`, `SOFTWARE`, `SYSTEM`, `DEFAULT` | the registry on disk. locked while windows runs, but readable from a boot disk, so physical access without BitLocker means credential exposure. the `RegBack` backup folder is usually empty on modern builds |
| `C:\Windows\System32\winevt\Logs\*.evtx` | event logs: `Security`, `System`, `Application`, powershell, sysmon | the main audit trail. file names like `Microsoft-Windows-PowerShell%4Operational.evtx` use `%4` because a filename can't contain `/` |
| `C:\Windows\System32\Tasks\` | XML definition of every scheduled task | the windows `cron`. new unknown tasks here are a top persistence check |
| `C:\Windows\System32\drivers\etc\hosts` | the hosts file | same job as `/etc/hosts`: a changed line silently redirects a domain |
| `C:\Windows\System32\drivers\*.sys` | kernel drivers | a malicious or vulnerable driver runs with kernel rights |
| `C:\Windows\System32\GroupPolicy\` | local group policy files | where local policy is stored |
| `C:\Windows\Prefetch\*.pf` | one file per program that ran: name, run count, last run times | proves a program executed even if it was deleted afterwards. clients only, servers usually disable it |
| `C:\Windows\AppCompat\Programs\Amcache.hve` | a hive with metadata of executed programs | another "it ran here" record for investigators |
| `C:\Windows\Temp` and `%TEMP%` | scratch files | where droppers and installers leave traces |
| `C:\Windows\Panther\` | setup logs and leftover answer files like `unattend.xml` | old unattended install files can contain admin credentials. delete them from images |
| `C:\Windows\Logs\CBS\CBS.log`, `C:\Windows\SoftwareDistribution` | servicing log and update cache | update failures and patch history |
| `C:\Windows\Minidump`, `C:\Windows\MEMORY.DMP` | crash dumps | memory contents at crash time, so treat them as secrets |
| `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp` | all users autostart folder | runs for every user at logon |

*Q: Which folder is the windows `cron`? A: `System32\Tasks`.*

### ntfs features linux doesn't have (and why they matter)

| feature | what it is | why i care |
| :-- | :-- | :-- |
| ACLs | a list of allow and deny entries per file, tied to SIDs, inherited from the parent | far more granular than `rwx`. also easier to misconfigure |
| alternate data streams (ADS) | extra hidden data attached to a file: `file.txt:stream` | windows marks downloads with a `Zone.Identifier` stream (Mark of the Web), which SmartScreen reads. malware can also hide payloads in streams |
| junctions, symlinks, hard links | `mklink /J`, `mklink`, `mklink /H` | junctions keep old paths alive. links can trick privileged programs into writing to the wrong place |
| 8.3 short names | `PROGRA~1` for `Program Files` | old DOS compatibility. visible in `dir /x` |
| `$MFT` and `$UsnJrnl` | NTFS's master file table and change journal | the metadata that forensics tools read to rebuild a timeline, even for deleted files |
| volume shadow copies | point in time snapshots | restore points, and a ransomware target |
| reserved names | `CON`, `NUL`, `COM1`, `LPT1` can't be filenames | DOS treated devices as names. `NUL` is the `/dev/null` of windows |
| case handling | preserved but not significant: `A.txt` equals `a.txt` | scripts and attackers can hit the same file by many spellings |

*Q: What is the Mark of the Web? A: A `Zone.Identifier` stream on downloads.*

### reading an ACL with icacls

`icacls C:\Windows\System32` prints one line per entry, shaped like this:

```text
NT SERVICE\TrustedInstaller:(F)
BUILTIN\Administrators:(M)
NT AUTHORITY\SYSTEM:(M)
BUILTIN\Users:(RX)
```

| piece | meaning |
| :-- | :-- |
| `NT SERVICE\TrustedInstaller` | the owner of OS files. it protects them even from administrators (Windows Resource Protection), so an admin must take ownership first |
| `(F)` | full control |
| `(M)` | modify: read, write, delete, but not change permissions |
| `(RX)` | read and execute. what regular users get on system folders |
| `(R)`, `(W)`, `(D)` | read, write, delete |
| `(OI)` | object inherit: applies to files inside |
| `(CI)` | container inherit: applies to subfolders |
| `(IO)` | inherit only: applies to children, not to this folder itself |
| `(NP)` | no propagate: only pass it one level down |
| deny beats allow | a deny entry wins over an allow entry, so a single `(DENY)` can lock out a group |

in a security review the dangerous line is any write right (`(M)`, `(W)`, `(F)`) for `BUILTIN\Users` or `Everyone` on a place where code runs.

*Q: Which wins, deny or allow? A: Deny.*

### commands for this section, broken down

| command | piece | what it does and why |
| :-- | :-- | :-- |
| `Get-ChildItem $env:APPDATA -Force \| Sort-Object LastWriteTime -Descending` | `$env:APPDATA` | the Roaming folder through the `env:` drive. a `$` starts a powershell variable |
|  | `-Force` | include hidden and system items |
|  | `\|` (pipe) | passes **objects**, not text. bash pipes bytes, so i parse columns with `cut` and `awk`. powershell hands over a real object with named properties, so i just name the property |
|  | `Sort-Object LastWriteTime` | sort by the `LastWriteTime` property. no column numbers needed |
|  | `-Descending` | newest first |
| `Get-Item .\file.exe -Stream *` | `-Stream *` | list all NTFS streams on the file. `*` is the wildcard for "every stream". if i see `Zone.Identifier`, it was downloaded |
| `Get-Content .\file.exe -Stream Zone.Identifier` | `-Stream` | read that one stream. it holds the `ZoneId` (3 means internet) |
| `icacls C:\Windows\Temp` | `icacls` | the newer version of the old `cacls` tool (the "i" is usually read as "improved", so that part is memorization): show or change ACLs. add `/grant`, `/deny`, `/reset` to change them |
| `dir /a` or `Get-ChildItem -Force` | `/a` | cmd's "attributes": include hidden and system files. cmd uses `/` for switches because DOS had no `-` convention |
| `Get-CimInstance Win32_Service \| Where-Object { $_.PathName -like '* *' -and $_.PathName -notlike '"*' } \| Select-Object Name, PathName` | `Get-CimInstance Win32_Service` | asks WMI for every service as an object |
|  | `Where-Object { ... }` | keeps objects where the block is true. `$_` is "the current object in the pipe" |
|  | `-like '* *'` | the path contains a space (`*` is a wildcard) |
|  | `-notlike '"*'` | and does not start with a quote. powershell uses word operators (`-eq`, `-like`, `-gt`) because `>` and `<` are already taken by redirection |
|  | `Select-Object Name, PathName` | print just those two properties |

the last command finds services with an unquoted path that contains a space, which is the next topic.

*Q: What does a powershell pipe pass? A: Objects, not text.*

### security angle: exploit and defend

- **unquoted service paths.** if a service runs `C:\Program Files\My App\svc.exe` with no quotes, windows tries the space separated pieces in order: `C:\Program.exe`, then `C:\Program Files\My.exe`, then the real file. a user who can write to `C:\` or `C:\Program Files\` plants a program at an earlier spot and it runs as SYSTEM when the service starts. defense: quote every service path, and keep write access off `C:\` and `Program Files`.
- **weak folder ACLs and DLL search order.** a folder that regular users can write, sitting in a service's load path or in `PATH`, lets them drop a malicious `.dll` or `.exe` that a privileged process loads. defense: audit with `icacls`, remove write rights for `Users`, and use signed binaries.
- **autostart folders and tasks.** `Startup` folders, `System32\Tasks` and the `Run` keys are the quiet ways to survive reboots. defense: baseline them, then alert on change (Sysmon, Autoruns, event logs).
- **streams and admin shares.** payloads can hide in ADS, and `\\host\C$` and `\\host\ADMIN$` let anyone with admin credentials move across machines. defense: scan for streams, limit local admin reuse, firewall SMB between workstations.
- **detect it:** `Security.evtx` records logon success (event 4624) and failure (4625); process creation (4688) shows up if the audit policy turns it on.

*Q: Why quote a service path? A: Windows tries each space split first.*

## 4. the windows registry

the registry is a hierarchical database: **keys** work like folders, **values** work like files (a name, a type and some data). windows stores almost all machine and user settings here, plus a lot of evidence about what happened. if `/etc` is linux's config folder, the registry is windows's `/etc` and `~/.config` fused into one database.

### the five root keys: two real, three shortcuts

| root key | short | what it really is |
| :-- | :-- | :-- |
| `HKEY_LOCAL_MACHINE` | HKLM | **real.** settings for this machine, all users |
| `HKEY_USERS` | HKU | **real.** one subtree per loaded user, named by SID |
| `HKEY_CURRENT_USER` | HKCU | shortcut to `HKU\<my SID>` |
| `HKEY_CLASSES_ROOT` | HKCR | merged view of `HKLM\Software\Classes` and `HKCU\Software\Classes`, and the user's copy wins |
| `HKEY_CURRENT_CONFIG` | HKCC | shortcut to the current hardware profile |

why shortcuts exist: a program doesn't know my SID, so `HKCU` lets it write "my settings" without looking up who i am. the same "user wins" merge in `HKCR` is what makes COM hijacking possible (see the security angle).

*Q: Which two root keys are real? A: HKLM and HKU.*

### hives: which file backs which key

a **hive** is a file that stores one subtree of the registry. the name is a joke on the structure (a bee hive made of cells), so it's memorization, not logic.

| registry path | file on disk | what it holds |
| :-- | :-- | :-- |
| `HKLM\SAM` | `C:\Windows\System32\config\SAM` | local accounts and password hashes (readable by SYSTEM only) |
| `HKLM\SECURITY` | `...\config\SECURITY` | security policy and LSA secrets |
| `HKLM\SOFTWARE` | `...\config\SOFTWARE` | installed software and OS settings |
| `HKLM\SYSTEM` | `...\config\SYSTEM` | boot config, drivers, services, hardware |
| `HKU\.DEFAULT` | `...\config\DEFAULT` | settings for the SYSTEM account |
| `HKLM\BCD00000000` | the `BCD` file on the boot partition | boot configuration data (replaces `boot.ini`) |
| `HKU\<SID>` (also HKCU) | `C:\Users\<name>\NTUSER.DAT` | one user's settings |
| `HKU\<SID>_Classes` | `...\AppData\Local\Microsoft\Windows\UsrClass.dat` | one user's file associations and shell data |
| `HKLM\HARDWARE` | none | built fresh at every boot, lives only in RAM |

next to each hive there are `.LOG1` and `.LOG2` files. those are transaction logs: windows writes changes there first so a crash can't leave the hive half written. they matter in forensics because they can hold changes that never reached the main file.

`CurrentControlSet` is a symlink inside `HKLM\SYSTEM` that normally points at `ControlSet001`. the others (`ControlSet002`) keep the last known good config, so a bad driver change can be rolled back at boot.

*Q: Which file holds my HKCU? A: `NTUSER.DAT`.*

### value types: why there are several

| type | holds | why it exists |
| :-- | :-- | :-- |
| `REG_SZ` | one string | the default text value |
| `REG_EXPAND_SZ` | a string with `%variables%` | windows expands `%SystemRoot%` when it reads it, so one value works on any install path |
| `REG_MULTI_SZ` | a list of strings | e.g. a service's dependencies |
| `REG_DWORD` | a 32 bit number | flags and switches (`0` or `1`), and small numbers like `Start` |
| `REG_QWORD` | a 64 bit number | sizes and timestamps |
| `REG_BINARY` | raw bytes | encoded data blobs, such as many forensic artifacts |

### the keys i check first

**autostart and persistence** (where code survives a reboot)

| key | what it does | why i care |
| :-- | :-- | :-- |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` and `RunOnce` | programs started at this user's logon | no admin needed to write here, so it's the most common persistence spot |
| `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` and `RunOnce` | programs started for every user | needs admin. the 32 bit twin lives under `HKLM\SOFTWARE\WOW6432Node\...` |
| `HKLM\SYSTEM\CurrentControlSet\Services\<name>` | one key per service: `ImagePath`, `Start`, `ObjectName` | `Start` values: 0 boot, 1 system, 2 automatic, 3 manual, 4 disabled. `ObjectName` is the account it runs as. a new service key is a classic backdoor |
| `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon` | `Shell` (normally `explorer.exe`) and `Userinit` | anything extra appended here runs at every logon |
| `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\<exe>` | a `Debugger` value | meant for debugging: windows launches the debugger instead of the exe. attackers use it to run their program whenever a tool starts |
| `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\AppInit_DLLs` | DLLs loaded into GUI processes | old trick, blocked when Secure Boot is on, but still worth a look |
| `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree` | index of scheduled tasks | matches the XML files in `System32\Tasks` |
| `HKCU\Software\Classes\CLSID\...` | per user COM class registrations | HKCU overrides HKLM in the merged `HKCR`, so a user can redirect a COM object with no admin rights |

**sysadmin settings**

| key | what it does | why i care |
| :-- | :-- | :-- |
| `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` (and the `WOW6432Node` twin) | one key per installed app | what "Programs and Features" reads. a software inventory in one place |
| `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion` | `ProductName`, `CurrentBuild`, `InstallDate` | quick OS and build check |
| `HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server` | `fDenyTSConnections` (1 = RDP off) | first place to check if RDP is enabled |
| `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Environment` and `HKCU\Environment` | system and user environment variables | where `PATH` is stored |
| `HKLM\SYSTEM\CurrentControlSet\Control\Lsa` | LSA settings, such as `RunAsPPL` | credential protection switches |
| `HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Shares` | defined SMB shares | what this machine exposes on the network |
| `HKLM\SOFTWARE\Policies\...` and `HKCU\Software\Policies\...` | where Group Policy writes its settings | policy values override user preferences, so this shows what an admin enforced |

**evidence of activity** (forensics and incident response)

| key | what it holds | why i care |
| :-- | :-- | :-- |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` | recently opened files | what the user touched |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU` | the Win+R history | commands typed into the Run box |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist` | run counts and last run of GUI programs | names are ROT13 encoded. that's obfuscation, not security, and any tool decodes it |
| `HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR` | every USB storage device ever plugged in | proves a device was connected, with vendor and serial |

*Q: Which Run key needs no admin rights? A: The `HKCU` one.*

### the 32 bit view

same trick as `SysWOW64`: a 32 bit program that opens `HKLM\SOFTWARE` is silently redirected to `HKLM\SOFTWARE\WOW6432Node`. so on a 64 bit machine one app can have two registry homes, and an audit has to check both.

### powershell and reg.exe: the commands, broken down

powershell treats the registry as a **drive** through a *provider*. `HKLM:` and `HKCU:` act like `C:`, so the same `Get-ChildItem` that lists folders lists registry keys. that's "everything is a file", windows style.

| command | piece | what it does and why |
| :-- | :-- | :-- |
| `Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run'` | `Get-ItemProperty` | reads the *values* of a key. `Get-ChildItem` lists subkeys, `Get-ItemProperty` reads the values inside one key. that split is the one to remember |
|  | `-Path` | which key to read. single quotes stop powershell from touching `$` or backticks in the string |
| `Get-ChildItem HKLM:\SYSTEM\CurrentControlSet\Services \| Get-ItemProperty \| Where-Object { $_.Start -eq 2 -and $_.ImagePath } \| Select-Object PSChildName, ImagePath` | `Get-ChildItem` | one object per service key |
|  | `Get-ItemProperty` (piped) | turns each key into an object holding its values |
|  | `$_.Start -eq 2` | keep automatic services (`-eq` is "equals") |
|  | `-and $_.ImagePath` | and the value exists (a non empty value counts as true) |
|  | `PSChildName` | powershell's name for "the key's own name", here the service name |
| `Set-ItemProperty -Path 'HKCU:\Software\Demo' -Name 'Flag' -Value 1` | `Set-ItemProperty` | changes a value. `New-ItemProperty` creates one, `Remove-ItemProperty` deletes one |
| `reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run /s` | `reg query` | the older cmd tool: show a key |
|  | `/s` | recursive: include all subkeys |
| `reg export HKCU\Software\Demo backup.reg` | `reg export` | saves a key to a text `.reg` file. i do this before every edit, because registry changes apply instantly and there is no undo |
| `reg load HKLM\TEMPHIVE C:\evidence\NTUSER.DAT` | `reg load` | mounts an offline hive file under a temporary name so i can browse it. used on disk images and dead machines |
| `reg unload HKLM\TEMPHIVE` | `reg unload` | releases it. forgetting this leaves the file locked |

the `reg` flags (`/s`, `/v`, `/t`, `/d`, `/f`) use the DOS style `/` because cmd predates `-` flags. `/v` names a value, `/t` gives its type, `/d` its data, `/f` skips the confirmation. they are mostly first letters, but treat them as memorization.

*Q: `Get-ChildItem` or `Get-ItemProperty` to read values? A: `Get-ItemProperty`.*

### security angle: exploit and defend

- **how it gets exploited:** persistence through `Run` keys, new service keys, `Winlogon` values, Image File Execution Options and per user COM hijacking. all of them are a single registry write, which makes them fast and quiet. the `HKCU` ones don't need admin rights at all.
- **how i defend it:** baseline the autostart keys (Sysinternals Autoruns shows them all in one view) and alert on change. Sysmon logs registry events: 12 for key create or delete, 13 for value set, 14 for rename. protect the offline hives with BitLocker so `SAM` and `SYSTEM` can't be read from a boot disk, and turn on LSA protection (`RunAsPPL`).

*Q: Why is `HKCU` persistence popular? A: It needs no admin rights.*

## 5. where software lives: apt, snap, flatpak and windows

the three linux package systems are three answers to one question: who owns the libraries? **apt** keeps one shared copy of each library for every app, so one security fix patches everything, but apps are tied to each other. **snap** and **flatpak** ship each app with its own dependencies inside a sandbox, so apps are independent, cost more disk, and are only as safe as their sandbox rules. windows is closer to the second model: one folder per app, with a newer sandboxed store model on top.

### the quick comparison

|  | apt (`.deb`) | snap | flatpak |
| :-- | :-- | :-- | :-- |
| who maintains it | debian and ubuntu | canonical (snap store) | flathub and other remotes |
| libraries | shared system wide | bundled in the snap | shared *runtimes* plus bundled extras |
| files live in | spread by type across `/usr`, `/etc`, `/var` | one compressed image, mounted at `/snap/<name>/<rev>` | one tree under `/var/lib/flatpak` or `~/.local/share/flatpak` |
| confinement | none: runs with my user's rights | AppArmor, seccomp and namespaces (unless installed as `--classic`) | bubblewrap sandbox with per app permissions |
| install needs | root | root | root for system, none with `--user` |
| updates | `apt upgrade` | automatic background refresh | `flatpak update` |
| my per app data | `~/.config/<app>` and friends | `~/snap/<name>/` | `~/.var/app/<app id>/` |

*Q: Which of the three shares libraries system wide? A: apt.*

### apt and dpkg: two layers

`dpkg` installs **one** `.deb` file and records it. `apt` is the dispatcher in front: it knows the repositories, downloads packages, solves dependencies, then hands each file to `dpkg`. two layers exist because `dpkg` has no idea where packages come from. similarly, `apt` is the friendly tool for interactive use, while `apt-get` is still preferred in scripts because its output format is more stable.

what happens on `sudo apt install nginx`:

1. the package index (from the last `apt update`) in `/var/lib/apt/lists/` tells apt what exists.
2. apt resolves dependencies and downloads `.deb` files into `/var/cache/apt/archives/`.
3. `dpkg` unpacks the files to their places (`/usr/sbin/nginx`, `/etc/nginx/`, ...), runs the package's maintainer scripts (`preinst`, `postinst`) **as root**, and records everything in `/var/lib/dpkg/`.

step 3 is why a package from a bad repository is dangerous: its install script runs as root.

| path | what it holds | why i care |
| :-- | :-- | :-- |
| `/etc/apt/sources.list.d/ubuntu.sources` | the default repositories, in the newer deb822 format | ubuntu 24.04 and later use this. a fresh 26.04 install may also have an old `/etc/apt/sources.list` that is empty or a stub. older debian still uses one line entries in `/etc/apt/sources.list`, so i check with `ls /etc/apt/sources.list.d` first |
| `/etc/apt/sources.list.d/*.sources` (or `.list`) | third party repositories | every file here is a source of root level code. audit it |
| `/etc/apt/keyrings/` and `/usr/share/keyrings/` | repository signing keys, referenced per repo by `Signed-By:` | the old global keyring (`apt-key`, now deprecated) trusted any key for any repo. a key tied to one repo limits the damage if it leaks |
| `/etc/apt/apt.conf.d/` | config snippets such as `20auto-upgrades`, `50unattended-upgrades` | numbered prefix = load order. the same trick as `/etc/sudoers.d` and `/etc/cron.d` |
| `/etc/apt/preferences.d/` | pinning rules (which repo wins) | the default priority is 500. a pin can force or block a version |
| `/var/lib/apt/lists/` | downloaded package indexes | what `apt update` refreshes |
| `/var/cache/apt/archives/` | downloaded `.deb` files | `apt clean` empties it |
| `/var/lib/dpkg/status` | every installed package and its state | the single source of truth for "what is installed" |
| `/var/lib/dpkg/info/` | per package `.list` (files), `.md5sums`, `postinst` and other scripts | how `dpkg -L` and `dpkg -V` know what a package owns |
| `/var/log/apt/history.log`, `/var/log/dpkg.log` | when things were installed or removed | the install timeline |
| `/usr/share/doc/<pkg>/` | docs, changelog, copyright | the license and changelog live next to the app |

*Q: Why does a bad apt repository mean root access? A: Install scripts run as root.*

### snap: a mounted image

a snap is one compressed read only file. the `/snap` folder is not where the files sit: it's where that image gets *mounted*. `df` and `mount` show it, and `du` on `/snap/<name>` overstates the disk use because it measures the unpacked view.

| path | what it is | why |
| :-- | :-- | :-- |
| `/var/lib/snapd/snaps/<name>_<rev>.snap` | the actual image file (squashfs, compressed) | the real storage cost of a snap |
| `/snap/<name>/<rev>` | the mount point of that image | read only, so the app can't modify its own files |
| `/snap/<name>/current` | symlink to the active revision | lets the launcher stay stable while revisions change |
| `/snap/bin` | launcher symlinks | on `PATH`, so `vlc` just works |
| `/var/snap/<name>/` | the snap's writable system data | system level state, kept separate from the read only image |
| `~/snap/<name>/` | the snap's data for my user | the per user twin of the one above |
| `/var/lib/snapd/` | snapd's state and assertions | the daemon's own database |

why `df -hT` is full of `/dev/loop` lines: every snap revision is a loop mounted squashfs image. each refresh leaves older revisions around for rollback, which is why `snap list --all` shows entries marked disabled.

*Q: Why are `/dev/loop` lines everywhere in `df`? A: Each snap revision is a loop mount.*

### flatpak: runtimes plus apps

flatpak splits an app into the **app** and a shared **runtime** (a base layer of libraries). many apps use the same runtime, so it's shared on disk. everything is stored in an OSTree repository, which dedupes files with hard links and makes rollbacks easy.

| path | what it is | why |
| :-- | :-- | :-- |
| `/var/lib/flatpak/` | system wide installation | available to all users, updated with admin rights |
| `~/.local/share/flatpak/` | per user installation (`--user`) | needs no admin rights and is invisible to other users |
| `.../app/<app id>/`, `.../runtime/` | installed apps and runtimes | each in `name/arch/branch` subfolders |
| `.../repo/` | the local OSTree repository | content addressed, so duplicate files cost nothing |
| `.../exports/` | launchers, icons and D-Bus files symlinked out of apps | how an app shows up in my menu |
| `~/.var/app/<app id>/` | that app's `data`, `config` and `cache` | the app's private "home"; it survives upgrades |
| `/etc/flatpak/installations.d/` | extra system installations (a second location) | for example, an app store on a separate disk |

the app id (`org.mozilla.firefox`) is reverse DNS naming: a convention that borrows a domain name so two vendors can't clash. memorize the shape, there is no deeper logic.

*Q: Where does a flatpak app keep its settings? A: `~/.var/app/<id>/config`.*

### commands, broken down

| command | piece | what it does and why |
| :-- | :-- | :-- |
| `dpkg -L bash` | `-L` | list files that a package installed. lowercase `-l` lists packages instead, and that pair is memorization, not a pattern |
| `dpkg -S /usr/bin/ls` | `-S` | "search": which package owns this file. answers "where did this binary come from?" |
| `dpkg -V openssh-server` | `-V` | "verify": compare installed files with the stored checksums. each output column is a check: a `5` means the md5 changed, `c` marks a config file (changes there are normal). no output means untouched |
| `apt policy nginx` | `policy` | installed vs candidate version, which repository each version comes from, and priorities (500 is default) |
| `apt why libssl3t64` | `why` | new in apt 3.1: shows what chain of packages pulled this one in. `t64` in the name marks the 64 bit `time_t` transition (it avoids the year 2038 problem) |
| `sudo apt update` then `sudo apt upgrade` | `update` | refreshes the *catalog* only. nothing on my system changes |
|  | `upgrade` | installs newer versions. it never removes a package, while `full-upgrade` may |
| `sudo apt purge nginx` | `purge` | removes the package **and** its config in `/etc`. plain `remove` keeps the config |
| `sudo apt modernize-sources` | `modernize-sources` | converts old `.list` files to the deb822 `.sources` format |
| `snap list --all` | `--all` | include disabled old revisions |
| `snap connections vlc` | `connections` | the permissions this snap has. a *plug* is something the app asks for, a *slot* is the system part that grants it |
| `flatpak list --app` | `--app` | show only applications, hide runtimes |
| `flatpak info --show-permissions org.mozilla.firefox` | `--show-permissions` | the sandbox rules: which folders, devices and sockets the app can reach |
| `flatpak override --user --nofilesystem=home <app id>` | `override` | my own permission change on top of the app's defaults, saved in an `overrides` folder next to the install |
|  | `--nofilesystem=home` | take away access to my home folder |

the `dpkg -V` check has a limit: the checksums sit on the same disk, so an attacker with root can edit both. for real integrity monitoring i need a database stored elsewhere (AIDE or a signed baseline).

*Q: What does `apt update` change on my system? A: Only the package catalog.*

### windows: where software lives

| installed by | files go to | metadata goes to | note |
| :-- | :-- | :-- | :-- |
| an `.msi` or `.exe` installer | `C:\Program Files\<vendor>\<app>` | `HKLM\...\Uninstall\<app>`, plus a cached copy in `C:\Windows\Installer` | the classic model. the cache is what repair uses |
| a per user installer | `%LOCALAPPDATA%\Programs\<app>` | `HKCU\...\Uninstall\<app>` | needs no admin rights, and it writes where users can write |
| the Microsoft Store (MSIX) | `C:\Program Files\WindowsApps\<package>` (locked, owned by TrustedInstaller) | per user data in `%LOCALAPPDATA%\Packages\<family name>` | sandboxed in an AppContainer, the closest thing to snap and flatpak |
| `winget` | whatever the vendor's installer does | same as above | `winget` is a front end: it fetches the vendor installer, it doesn't have its own file format like `.deb` |
| a portable app or a zip | wherever i unpack it | nothing | no registry trace, so the hardest to inventory |

the mapping that helps me think across both worlds:

| role | linux | windows |
| :-- | :-- | :-- |
| installer engine | `dpkg` | `msiexec` (Windows Installer) |
| front end with repositories | `apt` | `winget` |
| installed software database | `/var/lib/dpkg/status` | `Uninstall` registry keys and `C:\Windows\Installer` |
| app files | `/usr/bin`, `/usr/lib` | `C:\Program Files\<app>` |
| per user app data | `~/.config`, `~/snap`, `~/.var/app` | `AppData`, `%LOCALAPPDATA%\Packages` |
| sandboxed app format | snap, flatpak | MSIX (Store apps) |
| install history | `/var/log/dpkg.log` | Application event log (source `MsiInstaller`) |
| configured sources | `/etc/apt/sources.list.d` | `winget source list` |

| command | piece | what it does and why |
| :-- | :-- | :-- |
| `winget install --id Git.Git -e` | `--id Git.Git` | the exact package id, so there is no guessing between similar names |
|  | `-e` | short for `--exact`: match the id exactly instead of searching by substring |
| `Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall' \| Get-ItemProperty \| Select-Object DisplayName, DisplayVersion, Publisher` | the pipeline | an inventory straight from the registry. i repeat it for `WOW6432Node` (32 bit apps) and `HKCU:` (per user installs), or i miss software |
| `Get-AppxPackage -AllUsers \| Select-Object Name, InstallLocation` | `-AllUsers` | Store apps for every user. needs an admin shell |

i avoid `Get-CimInstance Win32_Product` for inventory: querying it makes windows run a consistency check on every installed MSI, which is slow and can trigger repairs.

*Q: Which windows tool resembles `apt`? A: `winget`, as a front end.*

### security angle: exploit and defend

- **how it gets exploited:** the package manager is a trusted channel into root. a malicious third party `.sources` file plus its key means every `apt upgrade` can install attacker code, and `postinst` scripts run as root. snaps installed with `--classic` and flatpaks granted `--filesystem=host` or `home` are barely sandboxed, so an app compromise reaches my files. on windows, per user installs and portable apps live outside `Program Files`, so they dodge folder ACLs and inventory tools.
- **how i defend it:** keep the repository list short and read `ls /etc/apt/sources.list.d`; pin each repo to its own key with `Signed-By`; check `apt policy` for surprises. review sandbox rights with `snap connections` and `flatpak info --show-permissions`, and tighten them with `flatpak override`. on windows, use application allow lists (AppLocker or WDAC) that block running code from `%LOCALAPPDATA%` and `%TEMP%` unless signed, and inventory all three places (HKLM, WOW6432Node, HKCU).

*Q: Why are `--classic` snaps riskier? A: They aren't confined by the sandbox.*

### sources

the snap, flatpak and apt facts above were checked on 2026-10-25 against these pages. the windows paths and registry keys come from my own knowledge, so i confirm them on a test VM with the commands in this section.

- [System snap directory (snapcraft docs)](https://snapcraft.io/docs/explanation/how-snaps-work/system-snap-directory/)
- [Flatpak filesystem wiki](https://github.com/flatpak/flatpak/wiki/Filesystem)
- [Complete APT guide for Ubuntu 26.04 (linuxconfig)](https://linuxconfig.org/complete-apt-package-management-guide-on-ubuntu-26-04)

## 6. file manager commands: bash and powershell

both shells do the same eight jobs: look, move around, create and delete, read, search, change permissions, check space, and combine commands. the real difference is the pipe: **bash pipes text** and **powershell pipes objects**. once that clicks, most of the syntax differences make sense.

### three ideas that make both shells easier

**1. who expands the wildcard.** in bash the *shell* expands `*` before the command runs. `ls *.txt` becomes `ls a.txt b.txt` and `ls` never sees the star. in cmd and powershell, the *command* (or the filesystem provider) handles the wildcard. the bash way means a file literally named `-rf` turns `rm *` into `rm -rf ...`. that is a real attack (wildcard injection), and the fix is `rm -- *` or `rm ./*`, where `--` means "options end here".

**2. text vs objects.** in bash, `ls -l` prints text and the next command has to cut it apart (`cut`, `awk`). in powershell, `Get-ChildItem` returns file objects with named properties (`Name`, `Length`, `LastWriteTime`), so the next command just names the property.

```text
bash:        ls -l | awk '{print $5, $9}'            # i must know the columns
powershell:  Get-ChildItem | Select-Object Length, Name  # i name the properties
```

**3. how i find out what a command does.**

| job | bash | powershell |
| :-- | :-- | :-- |
| read the manual | `man ls`, `ls --help` | `Get-Help Get-ChildItem -Examples` |
| search for a command by topic | `man -k permission` (or `apropos`) | `Get-Command *acl*` |
| what kind of thing is this word | `type ls` (alias, builtin or file?) | `Get-Command ls` |
| what can this output do | n/a (it's text) | `Get-ChildItem \| Get-Member` lists every property and method of the objects |
| search my history | `Ctrl+R`, `history` | `Ctrl+R`, `Get-History` |

*Q: Who expands `*` in bash? A: The shell, before the command runs.*

### 1. look and move around

| task | bash | powershell | flags and why |
| :-- | :-- | :-- | :-- |
| where am i | `pwd` | `Get-Location` (alias `pwd`) | "print working directory". the alias means muscle memory works in both |
| change folder | `cd /var/log`, `cd -`, `cd ~` | `Set-Location C:\Windows` (alias `cd`) | `cd -` jumps back to the previous folder, `~` is my home |
| list | `ls -lah` | `Get-ChildItem -Force` (aliases `ls`, `dir`, `gci`) | `-l` long, `-a` all (includes dotfiles), `-h` human sizes. short flags stack: `-lah` means `-l -a -h` |
| newest first | `ls -lt` | `Get-ChildItem \| Sort-Object LastWriteTime -Descending` | `-t` sorts by time. powershell sorts by property name instead |
| show a tree | `tree -L 2` | `Get-ChildItem -Recurse -Depth 1` | `-L 2` limits to 2 levels. `-Depth 1` goes one level below the start |
| details of one file | `stat file` | `Get-Item file \| Format-List *` | `stat` shows size, inode, permissions and the timestamps. `Format-List *` prints every property, one per line |
| what type is this | `file mystery.bin` | `Format-Hex .\mystery.bin -Count 16` | `file` reads the first bytes ("magic numbers"). windows programs start with `MZ` (the initials of an early engineer, Mark Zbikowski). linux programs start with `7F 45 4C 46`, which is DEL then `ELF` |
| real full path | `realpath f` | `Resolve-Path f` | resolves `.`, `..` and symlinks |

*Q: What do the first bytes `MZ` tell me? A: A windows executable.*

### 2. create, copy, move, delete

| task | bash | powershell | flags and why |
| :-- | :-- | :-- | :-- |
| make folders | `mkdir -p a/b/c` | `New-Item -ItemType Directory -Path a\b\c -Force` | `-p` = "parents": create missing parents and don't complain if it exists. `-Force` does the same here |
| create a file | `touch f` | `New-Item f -ItemType File` | `touch` really means "update the timestamp". creating the file is the side effect |
| copy | `cp -a src dst` | `Copy-Item src dst -Recurse` | `-a` = archive: recursive and keeps permissions, owner, times and symlinks. plain `-r` only recurses. powershell needs `-Recurse` for folders |
| move or rename | `mv old new` | `Move-Item old new` | one command for both, because a rename is a move inside the same folder |
| delete | `rm -r dir` | `Remove-Item dir -Recurse` | `-r` recursive. add `-f` or `-Force` to skip prompts. there is no recycle bin in either, so deletion is final |
| rehearse first | `rm -i f` (asks), or put `echo` in front | `Remove-Item f -WhatIf` | `-WhatIf` prints what *would* happen and does nothing. `-Confirm` asks for each item. bash has no universal twin |
| link | `ln -s target link` | `New-Item -ItemType SymbolicLink -Path link -Target target` | a symlink is a tiny file holding a path. `ln` without `-s` makes a hard link: a second name for the same data. the order is "existing thing first, new thing second", like `cp` |

*Q: Which flag lets `Remove-Item` rehearse safely? A: `-WhatIf`.*

### 3. read and slice files

| task | bash | powershell | flags and why |
| :-- | :-- | :-- | :-- |
| show a file | `cat f` | `Get-Content f` | `cat` = "concatenate": its original job was joining files, so printing one is a side use |
| scroll through | `less f` | `Get-Content f \| more` | `less` is a pun on `more` and lets me scroll back. inside it `/word` searches and `q` quits |
| first or last lines | `head -n 20 f`, `tail -n 20 f` | `Get-Content f -TotalCount 20`, `Get-Content f -Tail 20` | `-n` = number of lines |
| follow a log live | `tail -f /var/log/auth.log` | `Get-Content log.txt -Tail 20 -Wait` | `-f` = "follow", `-Wait` keeps the file open for new lines |
| count lines | `wc -l f` | `(Get-Content f).Count` | `wc` = word count, `-l` switches it to lines |
| count repeats | `sort f \| uniq -c \| sort -nr` | `Get-Content f \| Group-Object \| Sort-Object Count -Descending` | `uniq` only merges *neighbouring* duplicates, so i `sort` first. `-c` counts, `-nr` sorts numerically in reverse |
| pick a column | `cut -d: -f1 /etc/passwd` | `Get-Content p \| ForEach-Object { ($_ -split ':')[0] }` | `-d:` delimiter is a colon, `-f1` field one. powershell splits with `-split` and indexes from 0 |

*Q: Why `sort` before `uniq`? A: `uniq` only merges neighbouring lines.*

### 4. search

| task | bash | powershell | flags and why |
| :-- | :-- | :-- | :-- |
| by name | `find /etc -name '*.conf'` | `Get-ChildItem C:\ProgramData -Recurse -Filter *.xml` | `find` takes tests joined left to right. `-name` is case sensitive (`-iname` ignores case). i quote `'*.conf'` so the shell doesn't expand it first. `-Filter` is applied by the filesystem (fast), `-Include` by powershell afterwards (slower) |
| by age | `find /etc -type f -mtime -1` | `Get-ChildItem C:\Windows\System32\config -File -Force \| Where-Object LastWriteTime -gt (Get-Date).AddDays(-1)` | in `-mtime -1` the sign is a comparator: `-1` less than a day ago, `+1` more than a day ago, `1` exactly. units are 24 hours |
| by size | `find / -xdev -type f -size +100M` | `... \| Where-Object Length -gt 100MB` | `+100M` means bigger than 100 MiB. powershell understands the `MB` suffix as a number |
| by permission | `find / -xdev -perm -4000 -type f 2>/dev/null` | no setuid on windows | `-perm -4000` = "all of these bits are set" (setuid). `-perm /4000` = "any of them". `-xdev` stays on one filesystem (so it skips `/proc` and other mounts). `2>/dev/null` discards error messages |
| text inside files | `grep -rIn 'PermitRootLogin' /etc/ssh` | `Get-ChildItem C:\ProgramData -Recurse -Filter *.xml \| Select-String -Pattern 'password'` | `-r` recursive, `-I` skip binary files, `-n` line numbers. also `-i` ignore case, `-v` invert, `-c` count. the name `grep` comes from an old editor command, `g/re/p`: global, regular expression, print |
| act on results | `find . -type f -exec sha256sum {} +` or `find . -print0 \| xargs -0 sha256sum` | `\| ForEach-Object { Get-FileHash $_.FullName }` | `{}` stands for each found file, `+` batches many files into one run. `-print0` and `xargs -0` separate names with a null byte, so names with spaces or newlines can't break the command |
| fast name lookup | `locate name` | `Get-ChildItem -Recurse` is the slow built in way | `locate` searches an index built by `updatedb`: instant, but can be stale |

*Q: What does `-mtime -1` mean? A: Modified less than a day ago.*

### 5. permissions and ownership

| task | bash | powershell and windows tools | flags and why |
| :-- | :-- | :-- | :-- |
| look | `ls -l`, `getfacl f` | `Get-Acl f \| Format-List`, `icacls f` | `getfacl` shows the extra ACL entries linux also supports beyond `rwx` |
| change | `chmod 640 f`, `chmod u+x f`, `chmod -R go-w dir` | `icacls f /grant "BUILTIN\Users:(R)"` | symbolic form: who (`u` user, `g` group, `o` other, `a` all), operator (`+` add, `-` remove, `=` set), what (`r w x`). `-R` = recursive. numeric: r=4, w=2, x=1, added per group |
| owner | `sudo chown alice:staff f` | `takeown /F f` | `chown user:group`. on windows ownership is *taken*, not given: `takeown` makes me owner, then `icacls` edits the list |
| who am i | `id`, `groups` | `whoami /all` | `whoami /all` prints user, SID, group SIDs and privileges |
| default permissions | `umask` | inheritance from the parent folder | `umask 022` is a mask of bits to *remove* from 666 and 777, so new files are 644 and new folders 755 |
| what may i do | `sudo -l` | `whoami /priv` | `-l` = list the allowed commands |

*Q: What does `umask 022` do to a new file? A: It removes write for group and other.*

### 6. space, disks and mounts

| task | bash | powershell | flags and why |
| :-- | :-- | :-- | :-- |
| free space | `df -hT` | `Get-Volume` | `-h` human sizes, `-T` filesystem type |
| folder size | `du -sh /var/log/*` | `(Get-ChildItem C:\Users -Recurse -File -ErrorAction SilentlyContinue \| Measure-Object Length -Sum).Sum / 1GB` | `-s` = one summary line per argument, `-h` human. `Measure-Object` adds up a property |
| disks and partitions | `lsblk -f` | `Get-Disk`, `Get-Partition` | `-f` adds filesystem type, label and UUID |
| what is mounted | `findmnt` | `Get-PSDrive -PSProvider FileSystem` | `findmnt` draws the mount tree |

`df` and `du` can disagree, and the reason is useful: `df` asks the filesystem how much is used, `du` walks the files it can see. a file deleted while a process still holds it open keeps taking space but is invisible to `du`. on linux `lsof +L1` finds those.

*Q: Why can `df` and `du` disagree? A: Deleted files held open still use space.*

### 7. redirection and streams

every process starts with three channels: **0** input (stdin), **1** output (stdout), **2** errors (stderr). redirection points them somewhere else.

| i want to | bash | powershell | why |
| :-- | :-- | :-- | :-- |
| write output to a file | `cmd > f` | `cmd > f` | `>` overwrites |
| append | `cmd >> f` | `cmd >> f` | `>>` adds to the end |
| send errors to a file | `cmd 2> err.txt` | `cmd 2> err.txt` | `2` names the channel |
| merge errors into output | `cmd > f 2>&1` | `cmd *> f` | the `&` in `2>&1` means "this is a channel, not a file name". without it, `2>1` writes into a file called `1`. order matters: `> f 2>&1` works, `2>&1 > f` does not |
| throw output away | `cmd > /dev/null` | `cmd \| Out-Null` or `> $null` | `/dev/null` is the "black hole" file. the windows twin is `NUL` in cmd and `$null` in powershell |
| feed a file as input | `cmd < f` | `Get-Content f \| cmd` | powershell has no `<` yet |

powershell has six numbered streams: 1 success, 2 error, 3 warning, 4 verbose, 5 debug, 6 information. `*>` means "all of them".

*Q: What does `2>&1` do? A: Sends errors to wherever output goes.*

### 8. archives and hashes

| task | bash | powershell | flags and why |
| :-- | :-- | :-- | :-- |
| make an archive | `tar -czf out.tgz dir/` | `Compress-Archive -Path dir -DestinationPath out.zip` | `c` create, `z` gzip, `f` file name (it must be the last flag, because the name follows it). `tar` only bundles and `gzip` compresses: two jobs, two tools. zip does both |
| extract | `tar -xzf out.tgz -C /tmp` | `Expand-Archive out.zip -DestinationPath C:\Temp` | `x` extract, `-C` = change to this folder first |
| list contents | `tar -tzf out.tgz` | `Get-ChildItem` after expanding | `t` = table of contents |
| hash a file | `sha256sum f` | `Get-FileHash f -Algorithm SHA256` | compare with the vendor's published hash to detect tampering. `sha256sum -c list.sha256` checks a whole list |

*Q: Why list an archive before extracting? A: To check where files would land.*

### three one liners worth memorizing, piece by piece

**the ten biggest things under a folder**

| command | piece | what it does |
| :-- | :-- | :-- |
| `du -ah /var 2>/dev/null \| sort -rh \| head -n 10` | `du -ah /var` | size of every file and folder (`-a` all, `-h` human) |
|  | `2>/dev/null` | hide "permission denied" noise |
|  | `sort -rh` | `-r` reverse (biggest first), `-h` understands `K`, `M`, `G` |
|  | `head -n 10` | keep the first ten lines |
| `Get-ChildItem C:\Users -Recurse -File -ErrorAction SilentlyContinue \| Sort-Object Length -Descending \| Select-Object -First 10 FullName, Length` | `-ErrorAction SilentlyContinue` | the powershell twin of `2>/dev/null` |
|  | `Sort-Object Length -Descending` | sort by the `Length` property, biggest first |
|  | `Select-Object -First 10` | the twin of `head -n 10` |

**failed ssh logins per source address**

| command | piece | what it does |
| :-- | :-- | :-- |
| `grep 'Failed password' /var/log/auth.log \| awk '{print $(NF-3)}' \| sort \| uniq -c \| sort -nr \| head` | `grep 'Failed password'` | keep only failed attempts |
|  | `awk '{print $(NF-3)}'` | `awk` splits each line into fields. `NF` is the number of fields, so `$NF` is the last and `$(NF-3)` the fourth from the end, which is the IP in `... from 1.2.3.4 port 22 ssh2`. the parentheses matter: `$NF-3` would mean "the last field's value minus 3" |
|  | `sort \| uniq -c \| sort -nr` | group equal addresses, count them, biggest first |
| `Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} -MaxEvents 500 \| ForEach-Object { $_.Properties[19].Value } \| Group-Object \| Sort-Object Count -Descending \| Select-Object -First 10 Count, Name` | `-FilterHashtable @{...}` | ask the event log service to filter (fast), instead of fetching everything. `@{}` is a hashtable |
|  | `Id=4625` | event 4625 = failed logon |
|  | `$_.Properties[19].Value` | the source address field of that event. the index is memorization, so i verify it on my version with `$_.ToXml()`. needs an admin shell |

**every setuid file on the machine**

| command | piece | what it does |
| :-- | :-- | :-- |
| `find / -xdev -perm -4000 -type f 2>/dev/null` | `/` | start at the root |
|  | `-xdev` | don't cross into other filesystems |
|  | `-perm -4000` | the setuid bit is set (`4` in the special digit) |
|  | `-type f` | regular files only |
|  | `2>/dev/null` | hide errors |

the result should be a short, familiar list (`passwd`, `sudo`, `mount`). anything odd in it is a priority. windows has no setuid; the nearest idea is a service running as SYSTEM, covered in section 3.

*Q: What does `$(NF-3)` select in awk? A: The fourth field from the end.*

### security angle: exploit and defend

| mistake or trick | how it gets abused | how i defend |
| :-- | :-- | :-- |
| unquoted variables and wildcards | `rm -rf $DIR/*` with `DIR` empty becomes `rm -rf /*`. a file named `-rf` becomes a flag | quote variables (`"$DIR"`), use `--` and `./*`, and start scripts with `set -euo pipefail` |
| `set -euo pipefail` | n/a, it's the guard | `-e` exit on the first error, `-u` treat an unset variable as an error, `-o pipefail` fail if *any* stage of a pipe fails, not just the last |
| hidden shell history | a leading space can keep a command out of history (when `HISTCONTROL=ignorespace`), and `history -c` clears it | don't rely on history: use auditd and `sudo` logging |
| powershell history and logs | `ConsoleHost_history.txt` can be deleted, scripts can be obfuscated | turn on Script Block Logging (event 4104) and transcription, then send the logs elsewhere |
| filenames with spaces or newlines | break naive loops and `xargs` | `find -print0 \| xargs -0`, quote everything |
| dangerous bulk commands | one wrong `Remove-Item -Recurse -Force` is final | `-WhatIf` first, then run it for real |

*Q: What does `set -u` protect against? A: Using an unset variable by accident.*

## 7. structured breakdown (cheat sheet)

| concept | explanation | practical use and problem solved |
| :-- | :-- | :-- |
| one tree vs volumes | linux mounts every disk into one tree under `/`. windows gives each volume a letter on top of an NT object namespace | `findmnt` and `df -hT` show the linux tree, `Get-Volume` shows windows volumes |
| static vs variable | the FHS sorts folders by "does it change at runtime?" and "is it this machine only?" | put `/var` on its own partition so logs can't fill root. back up `/etc`, `/home`, `/var/lib` |
| config: text vs database | linux keeps config as plain text in `/etc`. windows uses the registry plus `ProgramData` | `grep` and diff `/etc` freely. `reg export` before every registry edit |
| user data and settings | `~/.config` and `~/.local/share` vs `AppData\Roaming` (follows me), `Local` (this machine), `LocalLow` (sandboxed apps) | know where an app keeps settings, caches and secrets |
| logs | `/var/log` text files plus the journal vs `System32\winevt\Logs\*.evtx` | `journalctl -u ssh`, `Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625}` |
| permissions | `rwx` for owner, group, other, plus setuid, setgid and sticky bits. windows uses ACL entries with inheritance, and TrustedInstaller owns OS files | `ls -l`, `chmod`, `icacls`. hunt for write rights granted to `Users` or `Everyone` |
| persistence spots | cron, systemd units, shell startup files, `authorized_keys` vs `Run` keys, Startup folders, services, scheduled tasks | baseline them and alert on change |
| registry hives | HKLM and HKU are real, the other roots are shortcuts. each hive is a file (`SAM`, `SYSTEM`, `NTUSER.DAT`) | `reg load` to inspect offline. BitLocker protects the hive files |
| 32 bit redirection | `SysWOW64` and `WOW6432Node` give 32 bit apps their own view of files and registry | audit both views or miss software and autostarts |
| NTFS streams | extra hidden data attached to a file. the `Zone.Identifier` stream marks downloads (Mark of the Web) | `Get-Item f -Stream *`. SmartScreen reads that mark |
| apt over dpkg | `dpkg` installs one `.deb`, `apt` finds repositories and solves dependencies | `dpkg -L`, `-S`, `-V` and `apt policy`, `apt why` |
| snap | a read only squashfs image mounted under `/snap`. data in `~/snap` | `snap list --all`, `snap connections <name>` |
| flatpak | an app plus a shared runtime in an OSTree repo. data in `~/.var/app` | `flatpak info --show-permissions`, `flatpak override` |
| windows software locations | Program Files (admin), `%LOCALAPPDATA%\Programs` (per user), `WindowsApps` (Store), portable apps (no trace) | inventory all of them: HKLM, `WOW6432Node`, HKCU and `Get-AppxPackage` |
| text vs object pipelines | bash passes bytes. powershell passes objects with named properties | use `Select-Object` and `Where-Object` instead of `awk` columns. `Get-Member` shows the properties |
| redirection and streams | channels 0 input, 1 output, 2 errors. powershell adds streams up to 6 | `cmd > f 2>&1`, `2>/dev/null`, `*>` |
| wildcard expansion | bash expands `*` before the command runs. windows commands expand their own | `rm -- *` and quoted variables prevent flag injection |

*Q: Which two registry roots are real? A: HKLM and HKU.*

## 8. feynman checks

if i can't answer two of these three without looking, i go back to the sections they come from.

**1. a ubuntu web server was breached. which places on disk do i check first, and what does each tell me?** (sections 2, 5 and 6)

- answer: `/var/log/auth.log` (or `journalctl`) for ssh and sudo activity. `~/.ssh/authorized_keys` for keys an attacker added. cron (`/etc/cron.d`, `/var/spool/cron/crontabs`) and systemd (`/etc/systemd/system`) for persistence. `/tmp` and `/dev/shm` for dropped payloads. `/etc/passwd` for a new uid 0 user. then `dpkg -V` to see which package files changed. the caveat: a root level attacker can also edit logs and checksums, so i compare against an offline baseline.

**2. a windows service runs `C:\Program Files\Acme Tools\agent.exe` with no quotes, and a normal user can create `C:\Program.exe`. what happens, how do i confirm it, and how do i fix it?** (sections 3 and 4)

- answer: when the service starts, windows tries `C:\Program.exe` first, because it splits the path at the first space. it would run the user's file as the service account, often SYSTEM. i confirm with `Get-CimInstance Win32_Service | Where-Object { $_.PathName -like '* *' -and $_.PathName -notlike '"*' }` and check `icacls C:\`. the fix: quote the `ImagePath` value under `HKLM\SYSTEM\CurrentControlSet\Services\<name>`, and remove write rights for `Users` on `C:\` and `Program Files`.

**3. i install an app as a snap, a flatpak and a Store app. for each one: where do the app files live, where does my data live, and how do i check what it may access?** (section 5)

- answer: **snap:** image in `/var/lib/snapd/snaps`, mounted at `/snap/<name>/<rev>`, my data in `~/snap/<name>`, permissions with `snap connections <name>`. **flatpak:** app and runtime under `/var/lib/flatpak` (or `~/.local/share/flatpak` with `--user`), my data in `~/.var/app/<id>`, permissions with `flatpak info --show-permissions <id>`. **Store app:** files in `C:\Program Files\WindowsApps\<package>` (locked), my data in `%LOCALAPPDATA%\Packages\<family>`, sandboxed in an AppContainer. a `--classic` snap or a flatpak with `--filesystem=host` skips most of the protection.

## 9. omitted chuff

i left these out on purpose so the notes stay learnable. each one is a good next round once this sticks.

| topic i cut | why it's a separate round |
| :-- | :-- |
| partitions, LVM, RAID, `fstab` tuning | storage layout is its own topic. here i only needed the mount idea |
| SELinux and AppArmor policy paths | needs the permission model from section 2 first |
| systemd internals (unit syntax, targets, timers) | big enough to fill its own notes. i only listed where units live |
| Active Directory paths (`SYSVOL`, `NTDS.dit`, GPO storage) | you picked client windows, but this is the next step for a windows sysadmin |
| deep forensic artifacts (shellbags, jump lists, SRUM, `$MFT` structure) | a whole discipline. i listed the first ones to check |
| `dnf`, `rpm`, `pacman` paths | outside the debian and ubuntu scope |
| containers (`/var/lib/docker`, overlay filesystems) | builds on the snap and flatpak idea of a mounted image |
| WSL paths (`\\wsl$`, `/mnt/c`) | where the two worlds in these notes meet |

*Q: Which cut topic is the next step for a windows sysadmin? A: Active Directory.*

## 10. watch it explained

ready to paste into youtube search. same order as the cheat sheet in section 7.

| concept | search query |
| :-- | :-- |
| one tree vs volumes | `linux filesystem hierarchy explained visually` |
| static vs variable | `linux FHS usr var etc explained` |
| config: text vs database | `windows registry explained hives keys values` |
| user data and settings | `windows AppData local roaming locallow explained` |
| logs | `linux var log files explained auth.log syslog` |
| permissions | `linux file permissions explained chmod setuid sticky bit` |
| persistence spots | `malware persistence techniques explained windows linux` |
| registry hives | `windows registry hive files SAM SYSTEM NTUSER explained` |
| 32 bit redirection | `SysWOW64 vs System32 WOW64 redirection explained` |
| NTFS streams | `NTFS alternate data streams zone identifier explained` |
| apt over dpkg | `apt vs dpkg difference explained` |
| snap | `how snap packages work squashfs mount explained` |
| flatpak | `flatpak runtimes sandbox how it works` |
| windows software locations | `where windows installs programs program files appdata msix` |
| text vs object pipelines | `powershell objects vs bash text pipeline explained` |
| redirection and streams | `bash redirection 2>&1 stdout stderr explained` |
| wildcard expansion | `bash globbing wildcard expansion shell explained` |
