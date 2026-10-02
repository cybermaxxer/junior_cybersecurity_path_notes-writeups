persona: penetration tester, linux/windows internals

tl;dr: local enum after getting a shell has two tracks: commands that dump system state, and files/registry hives that hold creds or privesc paths. linux: check suid bits, sudo rights, cron, bash history, writable paths. windows: check whoami privileges, autorun registry keys, SAM/SYSTEM hives, scheduled tasks, powershell history. linpeas/winpeas automate most of this but you should know the manual commands since they're what the tools wrap.

## linux: recon commands

| command | breakdown | why |
|---|---|---|
| `id` | no flags, just prints uid/gid/groups | tells you who you are and what groups might grant extra access (docker, sudo, adm groups are privesc paths) |
| `sudo -l` | `-l` = list. shows what the current user can run as another user via sudo | if you can run something as root without a password, that's often instant privesc |
| `find / -perm -4000 2>/dev/null` | `-perm` filters by permission bits. `4000` in octal is the SUID bit (the leading 4). `2>/dev/null` redirects stderr (fd 2) to the null device so permission denied spam doesn't flood your terminal | finds SUID binaries, which run with the owner's privileges (often root) regardless of who executes them. classic privesc hunting ground |
| `find / -writable -type f 2>/dev/null` | `-writable` checks write permission for current user, `-type f` restricts to regular files | world writable config/script files you can tamper with to escalate |
| `crontab -l` and `cat /etc/crontab` | `-l` = list the current user's cron jobs | scheduled jobs running as other users (often root) you might be able to hijack |
| `ss -tulpn` | `-t` tcp, `-u` udp, `-l` listening sockets only, `-p` show owning process, `-n` numeric (skip dns/service name resolution, faster and avoids leaking queries) | what's listening locally, might expose internal services not reachable from outside |
| `ps aux` | `a` = all users' processes, `u` = user oriented format (shows user, cpu, mem), `x` = include processes without a controlling terminal | spot running services, passwords sometimes visible in process args |
| `env` | no flags, dumps environment variables | creds/tokens get left in env vars more than people think |
| `uname -a` | `-a` = all info (kernel name, version, arch) | fingerprints the kernel for known privesc exploits |

note: `-perm -4000` the leading dash before 4000 means "at least these bits set," not "exactly." that's find's own syntax convention, not something to derive logically, just memorize that one.

## linux: files to check manually

| path | what it holds |
|---|---|
| `/etc/passwd` | user list, shells, uids. world readable by design |
| `/etc/shadow` | hashed passwords, root only normally, huge win if readable |
| `~/.bash_history`, `~/.zsh_history` | past commands, sometimes literal passwords typed into cli tools |
| `/etc/cron.d/`, `/etc/crontab`, `/var/spool/cron/` | scheduled tasks across the system, not just your user |
| `/etc/sudoers`, `/etc/sudoers.d/` | who can sudo what |
| ssh keys: `~/.ssh/id_rsa`, `authorized_keys` | lateral movement / persistence material |
| `/etc/os-release` | confirms distro/version for exploit matching |

## windows: recon commands (cmd + powershell)

| command | breakdown | why |
|---|---|---|
| `whoami /priv` | `/priv` is a switch (windows uses `/` not `-` by convention, just memorize that) that lists the privileges in your access token | privileges like `SeImpersonatePrivilege` or `SeBackupPrivilege` are direct privesc vectors (potato exploits etc) |
| `whoami /groups` | lists group memberships | tells you if you're in a group with extra rights you're not using yet |
| `systeminfo` | no flags, dumps os version, patches, hotfixes | fingerprints missing patches for known kernel exploits |
| `net user` / `net localgroup administrators` | `net` is the legacy user/group management tool, these are subcommands not flags | shows local accounts and who's actually an admin |
| `tasklist` | lists running processes, windows equivalent of `ps` | spot av/edr processes, services running as system |
| `netstat -ano` | `-a` all connections, `-n` numeric (no dns), `-o` show owning process id | same idea as `ss` on linux, map local listening services to pids |
| `Get-ChildItem -Recurse -Force` (powershell) | `-Recurse` walks subdirectories, `-Force` shows hidden/system files that are normally filtered out | manual filesystem crawl when you suspect hidden config/cred files |
| `reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` | `reg query` reads a registry key. `HKLM` = HKEY_LOCAL_MACHINE, the hive for machine wide settings (vs `HKCU` for the current user only) | the `Run` key is an autostart mechanism, windows launches anything listed here at login. common persistence and privesc hiding spot |

the `/` vs `-` flag style split isn't arbitrary logic you can derive, it's just legacy: cmd.exe tools inherited `/` from old dos conventions, powershell cmdlets use `-` because they followed unix style tooling conventions later. both exist side by side because windows kept backward compat.

## windows: files/registry to check manually

| location | what it holds |
|---|---|
| `C:\Windows\System32\config\SAM` + `SYSTEM` hive | local password hashes, needs SYSTEM level access to read live, classic offline dump target |
| `HKLM\SYSTEM\CurrentControlSet\Services` | service configs, if you can write to a service binary path with weak perms that's privesc |
| scheduled tasks: `schtasks /query /fo LIST /v` | same idea as cron, tasks running as other users |
| unattended install files: `C:\Windows\Panther\Unattend.xml`, `sysprep.inf` | sometimes contain plaintext local admin creds left over from deployment |
| powershell history: `%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt` | windows equivalent of bash_history, literal past commands |
| `HKCU\Software\...` and `HKLM\Software\...` run keys (both) | check both hives, not just one, since current user and machine wide autoruns differ |

mnemonic for the linux quick pass: **SPECS**: Sudo rights, Perms (suid), Env vars, Cron jobs, Shell history.
mnemonic for windows: **WARPS**: Whoami /priv, Autoruns (run keys), Registry services, PSReadLine history, Scheduled tasks.

both linpeas and winpeas exist specifically to automate every row above plus a lot more, they're the industry standard first script to run once you have a shell. worth knowing the manual commands first though since tools get flagged by edr and you'll need to do this by hand eventually.

feynman check:
1. why check `find / -perm -4000` specifically instead of just listing all files? → because SUID binaries run with the file owner's privileges regardless of who executes them, so a misconfigured SUID binary owned by root is a direct path to running code as root.
2. why does `netstat -ano`'s `-n` flag matter for recon speed/stealth? → it skips reverse dns lookups on every ip, which is both faster and avoids generating extra network traffic/queries that could get logged or noticed.
