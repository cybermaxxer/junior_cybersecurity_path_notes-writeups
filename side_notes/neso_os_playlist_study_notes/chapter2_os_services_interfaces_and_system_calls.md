# chapter 2: os services, interfaces and system calls

> **disclaimer:** i don't claim authorship of any of this. these are my study notes from the neso academy operating systems playlist. they complement the lectures and don't replace them. watch the videos.

**videos covered (playlist order):**

| # | video | link |
|---|---|---|
| 1 | Operating System Structure | https://www.youtube.com/watch?v=fvN98a_7AT4 |
| 2 | Operating System Services | https://www.youtube.com/watch?v=TQWERtMoKbI |
| 3 | User Operating System Interface | https://www.youtube.com/watch?v=psDpbWscPuE |
| 4 | System Calls | https://www.youtube.com/watch?v=lhToWeuWWfw |
| 5 | Types of System Calls | https://www.youtube.com/watch?v=EavqupVh8ls |
| 6 | System Programs | https://www.youtube.com/watch?v=UWDzhz8MVqc |

## table of contents
- [operating system structure](#operating-system-structure)
- [operating system services](#operating-system-services)
- [user operating system interface](#user-operating-system-interface)
- [system calls](#system-calls)
- [types of system calls](#types-of-system-calls)
- [system programs](#system-programs)
- [cheat sheet](#cheat-sheet)
- [interleaving](#interleaving)
- [feynman checks](#feynman-checks)
- [review questions](#review-questions)
- [security angle](#security-angle)
- [omitted chuff](#omitted-chuff)

---

## operating system structure

operating systems differ internally but all must do two things: **multiprogramming** and **time sharing (multitasking)**.

**multiprogramming.** a single user can't keep the cpu and i/o devices busy all the time. so the os keeps several jobs in memory and switches the cpu among them.

- the **job pool** holds all jobs entering the system. memory is limited (the example uses 512 mb), so only a subset (jobs 1 to 4) gets loaded *(see diagram, search anchor: "job pool and the memory layout")*.
- without multiprogramming: job 1 uses the cpu, goes to do i/o, and the cpu **sits idle**.
- with multiprogramming: job 1 goes to i/o, the cpu is handed to job 2; when job 2 waits, job 3 gets it, and so on.

analogy: a lawyer with many clients. while one case waits for paperwork, they work another instead of idling.

multiprogramming uses resources well but gives **no user interaction**.

**time sharing (multitasking).** the cpu switches among jobs so frequently that users can interact with each program while it runs.
- cpu speed is huge, human speed is slow. while user 1 thinks and types, the cpu runs user 2's job.
- each user feels they own the whole machine.
- built on **cpu scheduling plus multiprogramming**.
- each user has at least one program in memory. a program loaded into memory and executing is called a **process**.

*Q: what does time sharing add over multiprogramming? A: user interaction while jobs run.*

---

## operating system services

services the os provides to programs and users:

| service | what it covers |
|---|---|
| user interface | cli, gui |
| program execution | load a program into memory and run it |
| i/o operations | users can't touch devices directly; the os mediates |
| file system manipulation | create, delete, modify, search files; enforce access permissions |
| communications | processes exchange information on one computer or across a network |
| error detection | cpu, memory, i/o device errors (e.g. printer out of paper) handled without collapsing the system |
| resource allocation | give cpu, memory, files, i/o to processes so none starves forever and none hogs a resource forever |
| accounting | track which users use how much of which resources, for billing and statistics |
| protection and security | processes must not interfere with each other or the os; outsiders must authenticate (password) |

"a chain is only as strong as its weakest link": protection must cover the whole system.

*Q: who stops one process trampling another? A: the os protection service.*

---

## user operating system interface

two fundamental ways a user talks to the os:

1. **command line interface (cli) / command interpreter.** you type commands; you must remember them.
2. **graphical user interface (gui).** windows, menus, mouse; the most user friendly.

**command interpreter details:**
- in some systems it's part of the **kernel**; in others (windows xp, unix) a **special program**.
- when several exist they're called **shells**: bourne shell, c shell, bourne again shell (bash), korn shell.
- two ways it executes a command:
  1. the code for the command is **inside the interpreter** itself.
  2. the interpreter just **calls a separate program** that holds the code (unix style). then adding a command means adding a program, no interpreter change.

**demo: create and delete two directories**

| action | linux | windows cmd |
|---|---|---|
| where am i | `pwd` | (shows in prompt) |
| change directory | `cd Desktop` | `cd Desktop` |
| make directory | `mkdir my_file_1` | `mkdir my_file_1` |
| list | `ls` | `dir` |
| remove directory | `rm -rf my_file_1` | `rmdir my_file_1` |

`rm -rf`: `-r` recurse into directories, `-f` force with no prompts. in a gui you'd right click, new folder, then delete.

*Q: why do unix style shells call separate programs? A: new commands need no shell changes.*

---

## system calls

a **system call** is the interface to the services the os provides.

**two modes of execution:**

| | user mode | kernel mode |
|---|---|---|
| access | no direct memory or hardware access | direct access to memory and hardware (privileged) |
| if the program crashes | only that program fails | whole system can halt |

most programs run in user mode because it's safer. when a program needs a resource it makes a system call; the mode switches from user to kernel for that moment (a **context switch** between modes), then switches back.

a system call is a programmatic way for a program to request a service from the kernel. they're commonly available as routines in c and c++.

**worked example: copy file a to file b.** a long chain of system calls *(see diagram, search anchor: "sequence of system calls")*:

1. get the input file name: prompt on screen (call), accept keyboard input (call).
2. get the output file name: prompt (call), accept (call).
3. open input file. if it doesn't exist: abort (call).
4. create output file. if it already exists: abort (call).
5. loop: read from input (call), write to output (call), until read fails.
6. close output file (call).
7. print completion message (call).
8. terminate normally (call).

even this trivial job needs many calls; real programs make thousands per second.

*Q: why does a program switch to kernel mode on a system call? A: only the kernel may touch hardware.*

---

## types of system calls

five categories:

| category | examples |
|---|---|
| process control | end, abort; load, execute; create process, terminate process; get/set process attributes; wait for time; wait event, signal event; allocate and free memory |
| file manipulation | create, delete file; open, close; read, write, reposition; get/set file attributes |
| device management | request device, release device; read, write, reposition; get/set device attributes; logically attach or detach devices |
| information maintenance | get/set time or date; get/set system data; get/set process, file or device attributes |
| communications | create/delete communication connection; send, receive messages; transfer status information; attach or detach remote devices |

**logically attach:** the os recognizes a device as ready for use. a usb stick plugged in is physically attached; "safely remove hardware" logically detaches it while it's still physically in the port.

**mapping the copy example:** open, read, write, close are file calls; creating or ending the run uses process control.

*Q: usb "eject" while still plugged in is which kind of detach? A: logical.*

---

## system programs

system programs sit just above the os and below application programs. they give a convenient environment for program development and execution; some are simple user interfaces to system calls, others are complex.

| category | job |
|---|---|
| file management | create, delete, copy, rename, print, list files and directories |
| status information | date, time, free memory or disk space, number of users, logs, performance |
| file modification | text editors; search and transform file contents |
| programming language support | compilers, assemblers, debuggers, interpreters |
| program loading and execution | absolute and relocatable loaders, linkage editors, overlay loaders, debuggers |
| communications | virtual connections among processes, users and systems: browsing, email, remote login, file transfer |

**file management vs file modification:** management works on the outside of a file (create, rename, move); modification changes its inside (edit text).

**not system programs:** web browsers, word processors, spreadsheets, database systems and games often ship with the os but are **application programs**.

*Q: is a bundled browser a system program? A: no, it's an application program.*

---

## cheat sheet

| concept | explanation | real use |
|---|---|---|
| multiprogramming | several jobs in memory, cpu moves on when one waits | high cpu utilization |
| time sharing | switch fast so users interact | desktops, multi user servers |
| process | program loaded and executing | each app you open |
| os services (9) | ui, execution, i/o, files, comms, errors, allocation, accounting, protection | the checklist for "what does an os do" |
| cli vs gui | typed commands vs windows | scripting vs everyday use |
| shell | a command interpreter | bash, zsh |
| user vs kernel mode | restricted vs privileged | crash isolation |
| system call | request into the kernel | open, read, write |
| 5 call types | process, file, device, information, communication | classifying any call |
| system programs | utility layer above the os | editors, compilers, loaders |

## interleaving

| pair | difference |
|---|---|
| multiprogramming vs time sharing | utilization only vs plus user interaction |
| system call vs system program | the raw kernel interface vs a utility built on it |
| file management vs file modification | outside vs inside the file |
| system program vs application program | development/execution utilities vs end user tasks |
| user mode vs kernel mode | safe sandbox vs privileged with crash risk |

## feynman checks

1. **a user program wants to save a file. walk through what happens between the click and the disk.** the program runs in user mode, issues file system calls (create, open, write, close); each switches to kernel mode, the kernel checks permissions and drives the disk, then returns to user mode.
2. **why isn't it safe to let every program run in kernel mode, and what does that cost?** a bug in kernel mode can halt the whole machine and any program could touch any hardware or memory; the cost of user mode is a mode switch on each system call.
3. **a classmate says "the compiler is part of the os". argue both ways.** it's a system program (programming language support) shipped with the os, but it isn't the kernel and runs as an ordinary process using system calls.

## review questions

- (ch1) which hardware mechanism does a system call resemble, and how does it differ from a device interrupt?
- (ch1) why is a time sharing user able to interact between cpu bursts? (hint: human speed vs cpu speed)

## security angle

- **system call boundary:** attackers target kernel entry points (argument tampering, race conditions). defense: validate everything crossing from user to kernel; seccomp style filtering shrinks the usable call set.
- **path and shell injection:** if a program builds a command string from user input and hands it to a shell, input can smuggle extra commands. defense: avoid the shell, pass arguments as arrays.
- **detect:** unusual system call sequences (e.g. a text editor suddenly spawning a network connection).

## omitted chuff
- the list of common gui features, and repeated recap sentences.
- the long definition of each shell family name beyond the four listed.
- detail on the exact ubuntu and windows 7 desktops used in the demo.

*Q: which service stops two processes from reading each other's memory? A: protection and security.*
